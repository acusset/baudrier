# Reference: onClick → Link conversions

## Case 1 — Button supports `asChild` (Slot-based, e.g. shadcn/ui)

Keep a single rendered element: the Slot merges Link's `<a>` with Button's styling/props.

```tsx
// Before
'use client'
import { useRouter } from 'next/navigation'
import { Button } from '@/components/ui/button'

export function ViewProject({ id }: { id: string }) {
  const router = useRouter()
  return (
    <Button onClick={() => router.push(`/projects/${id}`)}>
      View project
    </Button>
  )
}
```

```tsx
// After — no 'use client' needed
import Link from 'next/link'
import { Button } from '@/components/ui/button'

export function ViewProject({ id }: { id: string }) {
  return (
    <Button asChild>
      <Link href={`/projects/${id}`}>View project</Link>
    </Button>
  )
}
```

## Case 2 — Plain wrapper Button or native `<button>` (no `asChild`)

Don't nest — a `<button>` inside an `<a>` is invalid HTML and React will warn about it. Replace the element itself with `Link`, carrying over `className`/variant and any other non-click props (drop `type="button"`, etc.). This applies the same way whether it's a native `<button>` or a Button component that just forwards `className` to one.

```tsx
// Before
'use client'
import { useRouter } from 'next/navigation'
import { Button } from '@/components/ui/legacy-button' // no asChild support

export function EditProfile() {
  const router = useRouter()
  return (
    <Button variant="outline" onClick={() => router.push('/profile/edit')}>
      Edit profile
    </Button>
  )
}
```

```tsx
// After — no 'use client' needed
import Link from 'next/link'
import { buttonVariants } from '@/components/ui/legacy-button'

export function EditProfile() {
  return (
    <Link href="/profile/edit" className={buttonVariants({ variant: 'outline' })}>
      Edit profile
    </Link>
  )
}
```

If there's no exported `buttonVariants` (or equivalent) to reuse, copy the className string the Button applies for that variant rather than inventing new styling. A plain `<button className="...">` converts the same way — swap the tag for `Link` and carry the className straight across.

## Mapping push/replace options to Link props

| onClick handler | Link |
|---|---|
| `router.push(path)` | `<Link href={path}>` |
| `router.replace(path)` | `<Link href={path} replace>` |
| `router.push(path, { scroll: false })` | `<Link href={path} scroll={false}>` |
| local `navigate(path)` wrapping `router.push` | same as `router.push(path)` |
| local `navigate(path, { replace: true })` wrapping `router.replace` | same as `router.replace(path)` |

`Link` renders its own `<a>` — it hasn't needed a child `<a>` tag since Next.js 13, so never wrap one.

## `'use client'` removal

After the conversion, grep the file for other client-only usage: other `useState`/`useEffect`/`useRouter`/`usePathname`/`useSearchParams` calls, other event handlers, or browser APIs (`window`, `localStorage`, etc.). If none remain, delete the `'use client'` directive at the top of the file — the component reverts to a Server Component.
