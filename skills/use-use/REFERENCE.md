# Reference: `await` → `use` conversions

## Case 1 — Server Component awaits, then passes data down

The page blocks on `getComments` even though only `<Comments>` needs it. Start the request, pass the promise, unwrap it in the child behind `<Suspense>`.

```tsx
// Before — app/post/[id]/page.tsx
export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);
  const comments = await getComments(params.id); // whole page waits for this

  return (
    <article>
      <PostBody post={post} />
      <Comments comments={comments} />
    </article>
  );
}
```

```tsx
// After — app/post/[id]/page.tsx
import { Suspense } from 'react';

export default async function Page({ params }: { params: { id: string } }) {
  const post = await getPost(params.id);
  const commentsPromise = getComments(params.id); // started, not awaited

  return (
    <article>
      <PostBody post={post} />
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </article>
  );
}
```

```tsx
// After — components/comments.tsx
'use client';
import { use } from 'react';

export function Comments({ commentsPromise }: { commentsPromise: Promise<Comment[]> }) {
  const comments = use(commentsPromise);
  return <ul>{comments.map((c) => <li key={c.id}>{c.body}</li>)}</ul>;
}
```

If `<Comments>` stays a Server Component, use `const comments = await commentsPromise;` there instead — `use` is only required across the client boundary.

## Case 2 — No Server Component: cache the client-created promise

A promise built in a Client Component's render body is new every render, so `use` suspends forever. Cache it outside render, keyed by its inputs.

```tsx
// Before — refetch loop, never resolves
'use client';
import { use } from 'react';

export function UserName({ id }: { id: string }) {
  const user = use(fetch(`/api/users/${id}`).then((r) => r.json()));
  return <span>{user.name}</span>;
}
```

```tsx
// After
'use client';
import { use } from 'react';

const cache = new Map<string, Promise<User>>();
function getUser(id: string) {
  if (!cache.has(id)) cache.set(id, fetch(`/api/users/${id}`).then((r) => r.json()));
  return cache.get(id)!;
}

export function UserName({ id }: { id: string }) {
  const user = use(getUser(id)); // stable per id
  return <span>{user.name}</span>;
}
```

In real code, prefer a Suspense-aware library (React Query `useSuspenseQuery`, SWR `{ suspense: true }`) — it owns the cache, retries, and invalidation. Hand-rolled caches are for throwaway cases.

## Case 3 — `use(Context)` after an early return

`useContext` can't be called conditionally; `use` can. This is the one case `use` replaces it.

```tsx
// Before — hook rules forbid this
function Sidebar({ collapsed }: { collapsed: boolean }) {
  const theme = useContext(ThemeContext);
  if (collapsed) return <CollapsedRail />;
  return <Panel style={{ background: theme.surface }}>…</Panel>;
}
```

```tsx
// After
import { use } from 'react';

function Sidebar({ collapsed }: { collapsed: boolean }) {
  if (collapsed) return <CollapsedRail />;
  const theme = use(ThemeContext); // legal after the early return
  return <Panel style={{ background: theme.surface }}>…</Panel>;
}
```

Reading context unconditionally at the top? Keep `useContext` — don't churn it.

## `await` vs `use`

| | `await` (Server Component) | `use` |
|---|---|---|
| Where | Server Components only | Server **or** Client Components |
| On resolve | continues from the `await` point | component re-renders from the top |
| Reads context | no | yes — `use(Context)` |
| Conditional / after early return | n/a | allowed |
| In `try/catch` | yes | **no** — use an error boundary |
| Streaming | subtree waits unless wrapped in `<Suspense>` | suspends at the `use` call, streams the rest |

Rule of thumb: **server data → `await`** and pass values down; **a promise unwrapped in a Client Component → `use`**; **context after a guard clause → `use`**.

## Suspense and error boundary placement

Every `use(promise)` needs a `<Suspense>` ancestor for the pending state and an error boundary for rejection (a rejected promise throws to the nearest one — `use` must not be wrapped in `try/catch`).

```tsx
<ErrorBoundary fallback={<CommentsError />}>
  <Suspense fallback={<CommentsSkeleton />}>
    <Comments commentsPromise={commentsPromise} />
  </Suspense>
</ErrorBoundary>
```

A route-level `error.tsx` / `loading.tsx` covers this for a whole page; wrap a narrower boundary when you want a smaller blast radius.
