# 35. Server state and caching

[Curriculum](../README.md)

Prerequisites: [React](../27-react/tutorial.md), [HTTP and Fetch](../34-http-fetch-and-rest/tutorial.md), and [client state](../33-client-state-management/tutorial.md).

## What you will learn

Distinguish freshness from retention, choose complete cache keys, and coordinate reads with writes. Use TanStack Query to demonstrate loading, background refresh, mutation status, and invalidation.

## Concepts and decisions

Server state is owned by another system and observed through requests. A cached response is a snapshot with a particular scope and age. It can become stale because time passed, another user changed data, or a local mutation succeeded. Client caching must therefore define identity, freshness, retention, refetch triggers, and conflict behavior.

A query key identifies the data requested. Include variables that change the result, such as resource ID, filter, page, locale, and relevant user scope. Two different requests sharing one key can display the wrong data. Conversely, unstable keys can prevent useful reuse. Avoid storing secrets in keys that may appear in debugging tools or logs.

Freshness and garbage collection answer different questions. `staleTime` controls how long data is considered fresh; `gcTime` controls how long inactive cache entries are retained in current TanStack Query versions. Stale data can still be displayed while a refetch occurs. A mutation changes remote data, after which related queries may need invalidation or an explicit cache update. See the [TanStack Query quick start](https://tanstack.com/query/latest/docs/framework/react/quick-start).

## Worked tutorial: cached notes with a mutation

Start with the runnable Vite React project from [chapter 27](../27-react/tutorial.md). In that project run:

```sh
npm install @tanstack/react-query
```

Replace `src/App.jsx` with the following complete example. The API is an in-memory simulation so no backend or external service is required:

```jsx
import { useState } from 'react';
import {
  QueryClient, QueryClientProvider, useMutation, useQuery, useQueryClient
} from '@tanstack/react-query';

const client = new QueryClient();
let database = [{ id: 'first', title: 'Understand cache identity' }];
const wait = () => new Promise(resolve => setTimeout(resolve, 500));

async function readNotes() {
  await wait();
  return structuredClone(database);
}
async function createNote(title) {
  await wait();
  const note = { id: crypto.randomUUID(), title };
  database = [...database, note];
  return note;
}

function Notes() {
  const [title, setTitle] = useState('');
  const queryClient = useQueryClient();
  const notes = useQuery({
    queryKey: ['notes'], queryFn: readNotes, staleTime: 30_000
  });
  const add = useMutation({
    mutationFn: createNote,
    onSuccess: async () => {
      setTitle('');
      await queryClient.invalidateQueries({ queryKey: ['notes'] });
    }
  });

  return (
    <main>
      <h1>Cached notes</h1>
      <form onSubmit={event => {
        event.preventDefault();
        if (title.trim() && !add.isPending) add.mutate(title.trim());
      }}>
        <label htmlFor="title">Note title</label>
        <input id="title" value={title} disabled={add.isPending}
          onChange={event => setTitle(event.target.value)} />
        <button disabled={add.isPending || !title.trim()}>
          {add.isPending ? 'Saving…' : 'Add note'}
        </button>
      </form>
      {add.isError && <p role="alert">Save failed: {add.error.message}</p>}
      {notes.isPending ? <p role="status">Loading notes…</p> :
        notes.isError ? <p role="alert">Load failed: {notes.error.message}</p> :
        <ul>{notes.data.map(note => <li key={note.id}>{note.title}</li>)}</ul>}
      {notes.isFetching && !notes.isPending && <p role="status">Refreshing…</p>}
      <button type="button" onClick={() => notes.refetch()}>Refresh</button>
    </main>
  );
}

export default function App() {
  return <QueryClientProvider client={client}><Notes /></QueryClientProvider>;
}
```

Run `npm run dev`. Expect an initial loading state, one note, and a new note after submission and refetch. Click Refresh to observe background fetching while the existing list stays visible. Reloading resets the simulated database; the example is not persistent storage. The query client is created outside rendering so ordinary rerenders do not recreate the cache.

## Choosing a mutation strategy

Invalidation is simple: declare related cached data stale and let a read confirm the server result. Direct cache updates can avoid another request when a mutation returns enough authoritative data. Optimistic updates immediately predict success, but need rollback, concurrency handling, and reconciliation with the response. Use them when latency matters and failures can be explained without corrupting the user's mental model.

If you optimistically append a note, preserve the previous cache value, cancel conflicting reads, assign a temporary identity, and replace that identity with the server's result. Two concurrent mutations can make naive rollback erase another successful update. A mutation log or more targeted rollback may be necessary.

## Cache layers and debugging

Browser HTTP caching, service-worker caching, CDN caching, and application query caching are different layers. Invalidating one does not automatically purge the others. Inspect response headers and network behavior before concluding that the application cache is broken. ETags and conditional requests can reduce transferred data while allowing the server to confirm freshness.

Forward a query's abort signal to real Fetch calls when cancellation is supported. The demonstration delay does not simulate transport cancellation. Clear or scope cached data when accounts change. Persisted caches need versioning, expiration, and a privacy policy; retaining every response indefinitely is rarely appropriate.

## Exercises and acceptance criteria

1. Add a filter to both the request and query key. Acceptance: two filters never reuse each other's result incorrectly.
2. Make one mutation fail deliberately. Acceptance: the entered title remains available for retry and the list contains no invented successful record.
3. Add optimistic creation. Acceptance: failure removes only the failed optimistic item, concurrent success is preserved, and final IDs match server IDs.

## References

- [TanStack Query: important defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults)
- [TanStack Query: query invalidation](https://tanstack.com/query/latest/docs/framework/react/guides/query-invalidation)
- [MDN: HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Caching)
