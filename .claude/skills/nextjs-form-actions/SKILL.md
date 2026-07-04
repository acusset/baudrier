---
name: nextjs-form-actions
description: Builds new forms and refactors existing ones in Next.js App Router apps to use Server Components, the Form component (next/form, falling back to React's <form>), and Server Actions — instead of client-side useState/useEffect + fetch/API-route submission. Use when creating a form in a Next.js app, or when asked to refactor a form off useEffect/onSubmit + fetch onto server actions, useActionState, or useFormStatus.
---

# Next.js Form Server Actions

## Quick start

```tsx
// app/posts/create/page.tsx — Server Component
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
  const title = formData.get('title')
  // ...persist...
  revalidatePath('/posts')
  redirect('/posts')
}
```

No API route, no `useState` for loading/error, no `useEffect`.

## Decision tree

- **Navigation/GET forms** (search, filters) → `action` is a **string** path. Use `next/form` — it prefetches the destination and does a client-side navigation.
- **Mutation forms** (create/update/delete) → `action` is a **function** (Server Action). Prefer `next/form` here too — when `action` is a function it behaves identically to React's `<form>`, so defaulting to it is free. Fall back to plain `<form>` only outside Next.js, or if you need `method`/`encType`/`target`/`key` on the form itself (next/form drops these).
- Need inline field errors or a pending state **without a full page reload** → wrap the action with `useActionState`, and read `pending` via `useFormStatus` in a child Submit button. This subtree becomes `'use client'`.
- Need the UI to update **before** the mutation resolves (e.g. optimistic list append) → add `useOptimistic` on top of the above.
- A plain server round-trip is acceptable (page redirect/re-render is fine) → skip all hooks; redirect with an error query param, or read fresh data straight from the re-rendered Server Component. Zero client JS for the form.

Start from "no hooks" and only add `useActionState`/`useFormStatus`/`useOptimistic` when the plain round-trip genuinely isn't good enough UX — each one pulls part of the tree into a Client Component.

## Frontend validation

Define the validation rules once, as a schema (e.g. Zod), in a module with no `'use client'`/`'use server'` directive so both sides can import it:

- **Server Action**: `schema.safeParse(formData)` (or per-field parsing) is the authoritative check — never trust the client. Return field errors through `useActionState`.
- **Client**: the same schema validates on `onBlur`/`onChange` for instant per-field feedback, before the round-trip. This needs a small amount of client state per field, but it's additive — the actual submission still goes through the Server Action, not a client-side fetch.

Keep native HTML5 constraints (`required`, `type="email"`, `pattern`, `minLength`) on the inputs regardless — they're a zero-JS baseline that works even before the schema-based validation hydrates.

Never skip the server-side check because the client already validated — client validation is always bypassable.

## Anti-patterns to replace

Signs a form needs this refactor:

- Controlled inputs (`useState` + `onChange`) whose only purpose is to be read in `onSubmit`
- `onSubmit` handler that calls `fetch`/`axios` against an internal API route
- `useState` tracking `isLoading`/`error`/`success` for that fetch
- `useEffect` that re-fetches or navigates after a "submitted" flag changes
- An `app/api/**/route.ts` whose only caller is this form

## Refactor workflow

1. Find the API route (or inline fetch) the form currently calls; identify what it does after validation (persist, redirect, revalidate).
2. Turn that logic into a Server Action: a function marked `'use server'`, either exported from its own `actions.ts` or declared inline inside a Server Component.
3. Read fields with `formData.get(name)` — don't thread individual field values in as separate arguments unless using `.bind()` to curry an id/context value ahead of the form data.
4. Validate inside the action against the shared schema (see "Frontend validation" above). Return a structured error value for expected validation failures; only `throw` for truly exceptional cases (an Error Boundary will catch those).
5. On success, call `revalidatePath`/`revalidateTag` for data the mutation affects, and `redirect()` if navigation should follow.
6. Wire the form: `<Form action={action}>` (or `<form action={action}>`), passing the action directly — no `onSubmit`, no `preventDefault`.
7. Drop the controlled-input state. Use `name` + `defaultValue` (uncontrolled) unless the value is genuinely needed for other UI (e.g. a live character count) — controlled inputs still submit fine via `FormData`.
8. If inline errors or non-navigating pending state are needed, wrap with `useActionState(action, initialState)` and render its returned state; this makes that component `'use client'`.
9. For a pending indicator, add a small child component using `useFormStatus` (must itself be `'use client'`; it only reads status from its parent `<form>`).
10. Add `useOptimistic` only if the mutation needs to feel instant before the server responds.
11. Delete the now-orphaned API route if nothing else calls it.

## Checklist

- [ ] No manual `fetch`/`axios` call backing this form's submission
- [ ] No `useEffect` firing off the back of a submission/loading flag
- [ ] Server Action function(s) marked `'use server'`
- [ ] Inputs are uncontrolled (`name` + `defaultValue`) unless controlled for a reason unrelated to submission
- [ ] Expected validation failures are returned from the action, not thrown
- [ ] Validation rules live in one shared schema imported by both the action and any client-side field checks
- [ ] Mutation calls `revalidatePath`/`revalidateTag` and/or `redirect()` as appropriate
- [ ] Pending UI comes from `useFormStatus`, not a local `isLoading` state
- [ ] Only the smallest necessary subtree is `'use client'` (e.g. the Submit button, not the whole page)
- [ ] Orphaned API route removed

See [REFERENCE.md](REFERENCE.md) for full templates: basic action, shared client/server validation schema, `useActionState` validation errors, `useFormStatus` submit button, `useOptimistic`, multiple submit actions (`formAction`), and a full before/after migration example.
