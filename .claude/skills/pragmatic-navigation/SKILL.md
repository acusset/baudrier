---
name: pragmatic-navigation
description: Replace Button/element onClick handlers that only call router.push, router.replace, or a navigate() wrapper with a Next.js Link composed with the button's styling. Use when a clickable element's onClick exists purely to navigate, or the user wants to stop using onClick navigation in favor of Link.
---

# Pragmatic Navigation

## Goal

`onClick={() => router.push(...)}` forces a component (and often its whole file) to be a Client Component for no real reason. A `<Link>` gets the same visual result with no JS handler, no `useRouter`, and lets the component stay a Server Component when nothing else needs the client.

## When to convert

The onClick's **only** job is navigation: a direct `router.push(...)`/`router.replace(...)` call, or a local `navigate()`/`goTo()` helper that itself just wraps one of those.

**Do not convert** — flag for manual review instead — if the handler also does any of:
- conditional logic deciding whether/where to navigate (auth checks, form validation)
- side effects before navigating (analytics, mutations, `preventDefault`, async awaits)
- anything besides a single push/replace call

## Workflow

1. **Find candidates**: grep for `router.push(`, `router.replace(`, and any local `navigate`/`goTo` helper, then check whether it's wired to an `onClick` on a `Button`/`button`/clickable element.
2. **Extract the destination** and any options (e.g. `{ scroll: false }` passed as a second arg to push/replace).
3. **Decide the composition** by inspecting the Button being used — see [REFERENCE.md](REFERENCE.md) for the three cases (asChild-capable Button, plain className-forwarding Button, native `<button>`) and full before/after examples.
4. **Map push/replace options to Link props** — see the mapping table in [REFERENCE.md](REFERENCE.md). `Link` doesn't need a child `<a>` (Next 13+) — never add one.
5. **Clean up the file**: remove the now-unused `useRouter`/`navigate` import and call. If nothing else in the file needs the client (no other `useState`/`useEffect`/event handlers/browser APIs), remove the `'use client'` directive so it reverts to a Server Component.
6. **Verify**: typecheck/build the touched files and spot-check that `Link`'s `href` matches what `router.push` used to receive (including any query params/dynamic segments).

## Notes

- If several onClick handlers in a file are pure navigation but others are legitimately interactive, only the file-level `'use client'` removal in step 5 is conditional on *all* client-only usage being gone — partial conversion is still worth doing.
- If the codebase has a shared `buttonVariants()`/`cva` export for the Button styles, prefer reusing it over duplicating class strings.
