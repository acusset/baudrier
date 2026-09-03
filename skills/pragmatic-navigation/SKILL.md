---
name: pragmatic-navigation
description: Replace Button/element onClick handlers that only call router.push, router.replace, or a navigate() wrapper with a Next.js Link composed with the button's styling. Use when a clickable element's onClick exists purely to navigate, or the user wants to stop using onClick navigation in favor of Link.
---

# Pragmatic Navigation

## Goal

`onClick={() => router.push(...)}` forces a component (often its whole file) to be a Client Component for nothing. A `<Link>` gives the same result with no handler, no `useRouter`, and lets the file stay a Server Component when nothing else needs the client.

## When to convert

Convert only when the onClick's **sole** job is navigation: a direct `router.push(...)`/`router.replace(...)`, or a local `navigate()`/`goTo()` helper that just wraps one.

**Flag for manual review instead** if the handler also does any of: conditional logic (auth checks, validation), side effects (analytics, mutations, `preventDefault`, `await`), or anything beyond a single push/replace call.

## Workflow

1. **Find candidates**: grep for `router.push(`, `router.replace(`, and any local `navigate`/`goTo` helper; confirm each is wired to an `onClick` on a `Button`/`button`/clickable element.
2. **Compose the Link** based on the Button in use — the three cases (asChild-capable Button, className-forwarding Button, native `<button>`), the push/replace-options→Link-props mapping, and before/after examples are all in [REFERENCE.md](REFERENCE.md).
3. **Clean up the file**: remove the now-unused `useRouter`/`navigate` import and call. If nothing else in the file needs the client (no other `useState`/`useEffect`/event handlers/browser APIs), remove the `'use client'` directive.
4. **Verify**: typecheck the touched files and check each `Link` `href` matches what `router.push` used to receive (query params and dynamic segments included).

## Notes

- Partial conversion is still worth doing: only the `'use client'` removal in step 3 is gated on *all* client-only usage being gone.
- Reuse a shared `buttonVariants()`/`cva` export for Button styling rather than duplicating class strings.
