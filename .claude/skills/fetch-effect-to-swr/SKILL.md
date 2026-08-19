---
name: fetch-effect-to-swr
description: Migrate client-side data fetching from a fetch call inside useEffect (with manual data/isLoading/error useState) to a custom hook built on SWR's useSWR. Use when a 'use client' component fetches data in a useEffect and tracks loading/error via useState, or the user mentions "SWR", "migrate to SWR", "replace useEffect fetch".
---

# Fetch-in-useEffect to SWR

## When to trigger

A Client Component (`'use client'`) that:
- Calls `fetch` inside a `useEffect`, and
- Tracks `data`, `isLoading`/`loading`, and `error` with separate `useState` calls, and
- Re-runs the effect when a dependency (id, query param, etc.) changes.

This is exactly the pattern SWR replaces: the effect, the three pieces of state, and any manual cleanup/cancellation/dedup logic all collapse into one `useSWR` call.

## Assumptions

- A shared `fetcher` utility exists (or gets created once) and is imported by every hook — don't redefine an inline fetcher per component.
- Scope is **read-only** (GET) fetching. Writes (`POST`/`PUT`/`DELETE`) via `useSWRMutation` or `mutate()` are out of scope for this skill.

## Quick start

Before:

```tsx
'use client'
function Profile({ userId }: { userId: string }) {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/user/${userId}`)
      .then((res) => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [userId]);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>failed to load</div>;
  return <div>hello {data.name}!</div>;
}
```

After:

```tsx
'use client'
import useSWR from 'swr';
import { fetcher } from '@/lib/fetcher';

function useUser(userId: string) {
  const { data, error, isLoading } = useSWR(`/api/user/${userId}`, fetcher);
  return { user: data, isLoading, error };
}

function Profile({ userId }: { userId: string }) {
  const { user, isLoading, error } = useUser(userId);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>failed to load</div>;
  return <div>hello {user.name}!</div>;
}
```

## Workflow

1. **Find the fetch**: the URL/params built inside the effect, and the effect's dependency array — those dependencies become the SWR key.
2. **Extract a hook** named after the resource (`useUser`, `useIssue`, ...) that calls `useSWR(key, fetcher)` and returns the fields the component actually needs (rename `data` to a domain name like `user`).
3. **Handle a key that isn't ready yet**: if the original effect guarded the fetch behind `if (!id) return`, pass a **function** as the key instead of a string — e.g. `() => \`/api/user/${userId}\`` — SWR skips the request until the function stops throwing/returning falsy. See [REFERENCE.md](REFERENCE.md) for the full pattern.
4. **Delete** the `useState` triple, the `useEffect`, and any manual cancellation/race-guard code (`isMounted` flags, `AbortController` cleanup) — SWR's caching and deduplication replace it.
5. **Update the component** to call the new hook and render from `isLoading`/`error`/the returned data.

## Reference

See [REFERENCE.md](REFERENCE.md) for a worked example with a dependent/conditional key, and notes on request deduplication.
