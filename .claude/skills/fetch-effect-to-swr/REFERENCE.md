# Reference

## Fetcher

SWR doesn't ship a fetcher — you provide one. The canonical shape from the [SWR docs](https://swr.vercel.app/docs/getting-started):

```ts
// lib/fetcher.ts
export const fetcher = (...args: Parameters<typeof fetch>) =>
  fetch(...args).then((res) => res.json());
```

Every generated hook imports this instead of redefining `fetch(...).then(...)` inline.

## Dependent / conditional fetching

Before — the effect guards on a value that might not exist yet (e.g. a selected id):

```tsx
'use client'
function IssueDetail({ issueId }: { issueId: string | undefined }) {
  const [issue, setIssue] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    if (!issueId) return; // nothing selected yet
    setIsLoading(true);
    fetch(`/api/issues/${issueId}`)
      .then((res) => res.json())
      .then(setIssue)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [issueId]);

  if (!issueId) return <div>Select an issue</div>;
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>failed to load</div>;
  return <div>{issue.title}</div>;
}
```

After — pass a **function** as the SWR key instead of a string. Per the [SWR docs on conditional fetching](https://swr.vercel.app/docs/conditional-fetching), SWR treats a thrown error or a falsy return from the key function as "dependencies not ready" and skips the request entirely (no `undefined` ever reaches the fetcher, no `if (!id) return` needed inside the hook):

```tsx
'use client'
import useSWR from 'swr';
import { fetcher } from '@/lib/fetcher';

function useIssue(issueId: string | undefined) {
  const { data, error, isLoading } = useSWR(
    () => `/api/issues/${issueId}`, // throws if issueId is undefined -> request skipped
    fetcher
  );
  return { issue: data, isLoading, error };
}

function IssueDetail({ issueId }: { issueId: string | undefined }) {
  const { issue, isLoading, error } = useIssue(issueId);

  if (!issueId) return <div>Select an issue</div>;
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>failed to load</div>;
  return <div>{issue.title}</div>;
}
```

## Deduplication note

Two components calling `useIssue(sameId)` at the same time share one request and one cache entry — SWR keys its cache by the resolved URL/key, so there's no need to lift the fetch up or pass data down as props just to avoid duplicate requests.

## Out of scope

Writes (`POST`/`PUT`/`DELETE`) are not covered here. Use `useSWRMutation` or the `mutate()` function returned by `useSWR` for those — that's a separate concern from the read-side migration this skill handles.
