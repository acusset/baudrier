---
name: use-use
description: Refactor a component that blocks on an awaited async function into a promise passed down and unwrapped with the `use` API behind a Suspense boundary. Use when a Server Component awaits data before rendering its subtree, a component chains `.then` on a promise in render, or the user mentions "use hook", "use()", "unwrap a promise", or "Suspense".
---

# Unwrapping promises with `use`

## Goal

Blocking on `const data = await getData()` in a Server Component stalls its whole subtree — nothing streams. Start the request without awaiting, pass the **promise** down as a prop, and unwrap it with `use(promise)` inside a `<Suspense>` so the rest of the tree renders now.

## When to trigger

- A Server Component `await`s `fn()` then renders a subtree that doesn't all need the result.
- A component calls `.then()`/`.catch()` on a promise in its render body.
- A parent `await`s only to hand resolved data to a child.

Skip it for `useEffect`+`useState` client fetching (use `fetch-effect-to-swr`), values every branch needs, or plain `useContext` reads.

## Workflow

1. **Find the blocking `await`/`.then`** and the value the subtree can render without.
2. **Drop `await`**, keep the call — `const p = getComments(id)` — and pass `p` down as a prop.
3. **Wrap the consumer in `<Suspense fallback={…}>`** with an error boundary above it — `use` rejects to the nearest boundary and must not sit in `try/catch`.
4. **In the consumer**, `const data = use(p)` and render from `data`. The promise must come from a Server Component prop, a module-level cache, or a Suspense-aware library — never one created during render.
5. **Verify**: the surrounding tree renders before the promise settles, the fallback shows, and a rejection hits the error boundary.

## Reference

See [REFERENCE.md](REFERENCE.md) for before/after conversions and an `await` vs `use` table.
