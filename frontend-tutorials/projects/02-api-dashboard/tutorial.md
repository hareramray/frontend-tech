# Project 02. Build a resilient API dashboard

[Projects](../README.md) · [Curriculum](../../README.md)

**Prerequisites:** JavaScript chapters 13–20, [fetch](../../34-http-fetch-and-rest/tutorial.md), [state](../../33-client-state-management/tutorial.md), [tables](../../69-search-tables-and-data-visualization/tutorial.md), and one framework if you choose to use one.

**Outcome:** a course catalog dashboard with loading, success, empty, error, and retry states; query and sort state in the URL; accessible results; and tests for races and failure recovery.

## Define a small contract

Start with a static fixture so you can build without an external service. Create `courses.json` in your application's public directory:

```json
{
  "courses": [
    { "id": "html", "title": "Semantic HTML", "level": "beginner", "lessons": 12 },
    { "id": "css", "title": "Responsive CSS", "level": "beginner", "lessons": 16 },
    { "id": "async", "title": "Async JavaScript", "level": "intermediate", "lessons": 18 },
    { "id": "perf", "title": "Performance profiling", "level": "advanced", "lessons": 10 }
  ]
}
```

The static fixture supports local filtering only. It does not become a server search endpoint when you add query parameters. Later, replace it with a development API using the same response contract and add genuine server-side filtering or pagination.

## Milestone 1: model data and states

Use a discriminated state such as `idle`, `loading`, `success`, and `error`. A successful response with an empty list is not a network error. Preserve a distinction between an initial load and a background refresh if old results remain visible.

Validate the response at the boundary. The following standalone module can be used in vanilla JavaScript or called from a framework's data layer:

```js
export async function loadCourses(signal) {
  const response = await fetch('/courses.json', { signal });
  if (!response.ok) throw new Error(`Could not load courses (${response.status})`);
  const data = await response.json();
  if (!data || !Array.isArray(data.courses)) throw new Error('Invalid course response');
  const valid = data.courses.every(course =>
    course && typeof course.id === 'string' &&
    typeof course.title === 'string' &&
    ['beginner', 'intermediate', 'advanced'].includes(course.level) &&
    Number.isInteger(course.lessons) && course.lessons >= 0
  );
  if (!valid) throw new Error('Invalid course record');
  return data.courses;
}
```

**Check:** removing the fixture produces a visible failure and retry action. Invalid JSON and an invalid record produce distinct developer evidence without dumping raw server responses into the user interface.

## Milestone 2: build the interaction

Create a labeled search input, level select, sort select, result count, and semantic results table or list. Keep source records separate from derived filtered results. Apply filtering before pagination. Use numeric comparison for lesson counts and locale-aware comparison for titles. Show a clear empty state with a reset-filters action.

Store non-sensitive view state in `URLSearchParams`, for example `?q=css&level=beginner&sort=title`. Parse unknown values defensively. Update the URL without losing unrelated parameters. On back/forward navigation, restore controls and results from the URL. Choose `replaceState` for frequent typing changes and `pushState` for meaningful navigable transitions according to the experience you want.

**Check:** copy a filtered URL into another tab and obtain the same view. Browser Back restores the prior intended state. A typo in a URL parameter does not crash the page.

## Milestone 3: handle races

When you move to remote search, a slow earlier response can arrive after a newer response. Abort the old fetch when starting a new request and use a request identifier if the data path includes operations that cannot be aborted. Treat cancellation as expected control flow. Do not announce it as a user-facing error.

Use a cache library only after you understand the state model, or implement a deliberately small cache keyed by all server-relevant inputs. Keep view state separate from server data. A search response for page one must not satisfy a cache lookup for page two.

**Check:** delay the first request, quickly enter a second query, then allow both responses to finish. Only the second query's result should appear. Unmounting or navigating away must not leave subscriptions or repeated requests behind.

## Milestone 4: test and observe

Write unit tests for filter/sort functions. Use component tests for labels, empty states, and retry behavior. Use an end-to-end test for URL sharing and back navigation. Mock the network at a deliberate boundary rather than assuming a public API is always available.

Measure a realistic large dataset before adding virtualization. If the full dataset becomes too large, move filtering, sorting, and pagination to the server together. Document whether totals describe all results or only a loaded page. Add loading announcements that are useful without announcing every keystroke excessively.

**Completion criteria:** every state is visible and testable; stale results cannot win; filters survive reload; the interface works by keyboard; source data remains immutable; numeric sorting is correct; and tests cover both a successful request and at least two failure paths.

**Stretch work:** add selected-row export with a deliberate CSV policy, saved filters, column visibility, optimistic bookmarking with rollback, and a documented API contract. Avoid mixing in authentication until the public-data behavior is reliable.
