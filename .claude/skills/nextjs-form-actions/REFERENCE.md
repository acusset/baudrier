# Reference: Templates

## Basic Server Action, no client hooks

```tsx
// app/posts/create/page.tsx
import Form from 'next/form'
import { createPost } from './actions'

export default function Page() {
  return (
    <Form action={createPost}>
      <input name="title" />
      <button type="submit">Create Post</button>
    </Form>
  )
}
```

```ts
// app/posts/create/actions.ts
'use server'
import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string
  const post = await db.post.create({ data: { title } })
  revalidatePath('/posts')
  redirect(`/posts/${post.id}`)
}
```

Zero client JS. Works before hydration finishes; degrades to a full navigation without JS at all.

## Passing extra context with `.bind()`

Use when the action needs an id/value that isn't a form field (e.g. from props or a route param):

```tsx
// components/add-to-cart.tsx
import { updateCart } from '@/lib/actions'

export function AddToCart({ productId }: { productId: string }) {
  const addProductToCart = updateCart.bind(null, productId)
  return (
    <form action={addProductToCart}>
      <button type="submit">Add to Cart</button>
    </form>
  )
}
```

```ts
'use server'
export async function updateCart(productId: string, formData: FormData) {
  await db.cart.add(productId)
}
```

## Shared client/server validation schema

Define the schema once, with no `'use server'`/`'use client'` directive, so it can be imported from both sides.

```ts
// app/posts/create/schema.ts
import { z } from 'zod'

export const postSchema = z.object({
  title: z.string().min(1, 'Title is required').max(120, 'Title is too long'),
})

export type PostFieldErrors = Partial<Record<keyof z.infer<typeof postSchema>, string>>
```

Server Action — authoritative check, runs regardless of what the client did:

```ts
// app/posts/create/actions.ts
'use server'
import { redirect } from 'next/navigation'
import { revalidatePath } from 'next/cache'
import { postSchema, type PostFieldErrors } from './schema'

type State = { errors: PostFieldErrors }

export async function createPost(prevState: State, formData: FormData): Promise<State> {
  const result = postSchema.safeParse({ title: formData.get('title') })
  if (!result.success) {
    return { errors: result.error.flatten().fieldErrors as PostFieldErrors }
  }
  const post = await db.post.create({ data: result.data })
  revalidatePath('/posts')
  redirect(`/posts/${post.id}`)
}
```

Client component — `onBlur` re-parses the same schema for instant per-field feedback, but submission still goes through the Server Action, not a client fetch:

```tsx
// app/posts/create/create-post-form.tsx
'use client'
import { useActionState, useState } from 'react'
import { createPost } from './actions'
import { postSchema, type PostFieldErrors } from './schema'
import { SubmitButton } from '@/components/submit-button'

const initialState = { errors: {} as PostFieldErrors }

export function CreatePostForm() {
  const [state, action] = useActionState(createPost, initialState)
  const [titleError, setTitleError] = useState<string>()

  function validateTitle(value: string) {
    const result = postSchema.shape.title.safeParse(value)
    setTitleError(result.success ? undefined : result.error.issues[0]?.message)
  }

  return (
    <form action={action}>
      <input name="title" required maxLength={120} onBlur={(e) => validateTitle(e.target.value)} />
      {(titleError ?? state.errors.title) && <p>{titleError ?? state.errors.title}</p>}
      <SubmitButton />
    </form>
  )
}
```

`required`/`maxLength` give a zero-JS baseline; `onBlur` gives instant feedback once hydrated; `state.errors` is the authoritative fallback covering anything the client skipped (JS disabled, stale bundle, bypassed input).

## Inline field errors: `useActionState`

The action's return value becomes the hook's state — use it for validation errors instead of a manual `error` `useState`.

```tsx
// components/signup-form.tsx
'use client'
import { useActionState } from 'react'
import { signup } from './actions'

const initialState = { error: null }

export function SignupForm() {
  const [state, signupAction] = useActionState(signup, initialState)
  return (
    <form action={signupAction}>
      <label htmlFor="email">Email</label>
      <input name="email" id="email" />
      <button type="submit">Sign up</button>
      {state.error && <p role="alert">{state.error}</p>}
    </form>
  )
}
```

```ts
// actions.ts
'use server'
export async function signup(prevState: { error: string | null }, formData: FormData) {
  const email = formData.get('email') as string
  if (!email.includes('@')) {
    return { error: 'Enter a valid email' }
  }
  await signUpNewUser(email)
  return { error: null }
}
```

Note the action's signature changes to `(prevState, formData)` once passed through `useActionState`.

## Pending state: `useFormStatus`

`useFormStatus` reads status from the nearest parent `<form>` — it must be called from a **child** component of the form, not the component that renders the form itself.

```tsx
// components/submit-button.tsx
'use client'
import { useFormStatus } from 'react-dom'

export function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting…' : 'Submit'}
    </button>
  )
}
```

The page/form itself can stay a Server Component; only `SubmitButton` needs `'use client'`.

## Optimistic updates: `useOptimistic`

Use when the list/value should update before the server round-trip resolves.

```tsx
'use client'
import { useOptimistic, useRef } from 'react'
import { sendMessage } from './actions'

export function Thread({ messages }: { messages: Message[] }) {
  const formRef = useRef<HTMLFormElement>(null)
  const [optimisticMessages, addOptimistic] = useOptimistic(
    messages,
    (state, newMessage: string) => [...state, { text: newMessage, sending: true }]
  )

  async function formAction(formData: FormData) {
    addOptimistic(formData.get('message') as string)
    formRef.current?.reset()
    await sendMessage(formData)
  }

  return (
    <>
      {optimisticMessages.map((m, i) => (
        <div key={i}>{m.text}{m.sending && <small> (sending…)</small>}</div>
      ))}
      <form action={formAction} ref={formRef}>
        <input name="message" />
        <button type="submit">Send</button>
      </form>
    </>
  )
}
```

## Multiple submit actions on one form

Use `formAction` on individual buttons to override the form's `action` per-button (e.g. "Save draft" vs "Publish"):

```tsx
import { publish, saveDraft } from './actions'

export function Editor() {
  return (
    <form action={publish}>
      <textarea name="content" />
      <button type="submit">Publish</button>
      <button formAction={saveDraft}>Save draft</button>
    </form>
  )
}
```

## Full migration: what changes

A typical "before" — controlled `title` state, `handleSubmit` calling `fetch('/api/posts', ...)`, `isLoading`/`error` state, `router.push` on success, backed by an `app/api/posts/route.ts` — migrates by combining the patterns above: the API route's body becomes the `'use server'` action (Basic template), controlled input becomes `name` + `defaultValue`, `isLoading` is replaced by a `SubmitButton` using `useFormStatus`, and `error` is replaced by `useActionState`'s returned state (Inline field errors template). The API route is then deleted.
