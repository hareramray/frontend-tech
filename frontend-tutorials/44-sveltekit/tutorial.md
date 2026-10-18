# SvelteKit: routes, server loads, and progressive enhancement

[Curriculum](../README.md)

Prerequisites: [Svelte](../30-svelte/tutorial.md), [TypeScript](../20-typescript/tutorial.md), and [rendering strategies](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

## Learning outcomes

Create a routed Svelte application, pass server-loaded data into a page, distinguish universal and server-only modules, and plan an enhanced form without breaking ordinary browser behavior. Svelte handles component reactivity; SvelteKit adds the application routing, loading, server, and deployment layers.

## Core concepts

SvelteKit maps `src/routes` to URLs. `+page.svelte` renders a page, `+layout.svelte` wraps nested routes, and `+server.ts` exposes an HTTP endpoint. A `+page.server.ts` load function runs only on the server, while a universal `+page.ts` load can participate in server rendering and browser navigation. Returned load data is available to the page and can be serialized to the client, so a server-only load function does not make its returned values secret. [Routing](https://svelte.dev/docs/kit/routing), [loading data](https://svelte.dev/docs/kit/load)

Page options can control server rendering, client rendering, and prerendering. An adapter prepares the build for a specific deployment environment. Choosing a static adapter is an architectural decision: request-time authentication and arbitrary backend actions require an appropriate server runtime.

## Worked tutorial: a searchable lesson directory

1. In your projects directory, run the official scaffolder:

```sh
npx sv create lesson-directory
cd lesson-directory
npm install
```

Choose a minimal SvelteKit project, TypeScript, and npm. Keep generated configuration. The CLI may offer optional integrations; none are needed for this example. [Creating a SvelteKit project](https://svelte.dev/docs/kit/creating-a-project)

2. Create `src/routes/+page.server.ts`:

```ts
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async () => {
  return {
    lessons: [
      { id: 'markup', title: 'Markup', level: 'Beginner' },
      { id: 'events', title: 'Events', level: 'Intermediate' },
      { id: 'caching', title: 'Caching', level: 'Advanced' },
    ],
  };
};
```

3. Replace `src/routes/+page.svelte`:

```svelte
<script lang="ts">
  import type { PageData } from './$types';
  let { data }: { data: PageData } = $props();
  let query = $state('');
  let filtered = $derived(data.lessons.filter(lesson =>
    lesson.title.toLowerCase().includes(query.trim().toLowerCase())
  ));
</script>

<svelte:head>
  <title>Lesson directory</title>
  <meta name="description" content="Find your next frontend lesson.">
</svelte:head>

<main>
  <h1>Lesson directory</h1>
  <label>Search lessons <input type="search" bind:value={query}></label>
  <p role="status">{filtered.length} lessons found</p>
  <ul>
    {#each filtered as lesson (lesson.id)}
      <li>{lesson.title} — {lesson.level}</li>
    {/each}
  </ul>
  {#if filtered.length === 0}<p>Try a different search.</p>{/if}
</main>
```

4. Run `npm run dev` and visit the address printed by the server. Search for `event`; exactly one row should remain. Clear the input and all three rows should return. Run `npm run check` if the generated project includes that script, followed by `npm run build`.
5. Inspect the initial document response. The lesson names should be present before the client executes. Disable JavaScript and reload: reading the directory still works, while this local filter requires JavaScript.

## Forms, state, and server boundaries

For writes, SvelteKit form actions can handle ordinary POST submissions. Begin with a working HTML form and server validation, then use `use:enhance` when the form meets its documented action requirements. Enhanced submissions should retain meaningful pending, success, and field-error states. A disabled button is useful feedback but does not make a backend mutation idempotent.

Use server hooks and request-local context to establish authenticated identity, then authorize each load, action, or endpoint that exposes protected data. Avoid storing a current user in a mutable module-level variable: a long-lived server process serves many users. Only pass the minimal fields a page needs. Consult [authentication](../39-authentication-and-authorization/tutorial.md) before connecting a real account system.

Load functions should describe data dependencies. Revalidation after a mutation should target affected data rather than force unrelated page work. For complex navigation, define what happens to unsaved edits, focus, and scroll position. Application state can survive navigation even when a developer expects a component to be recreated.

## Pitfalls and debugging

`window` and `document` are unavailable during server execution. Put suitable browser work into lifecycle callbacks and keep initial output stable. Locale formatting and random values can cause hydration disagreement. When a request fails, check whether it came from a load function, action, or endpoint; each has a different user-visible error contract.

Type generation depends on SvelteKit's project tooling. If `./$types` appears missing after adding a route, run the generated prepare/check workflow or restart the development tooling before inventing manual replacement types. Read adapter documentation before deploying filesystem access, cookies, or streaming behavior to another runtime.

## Exercises and acceptance criteria

- Add a route for one lesson using a dynamic segment. An unknown ID must produce an actual not-found response.
- Replace the local search with a GET form using a query parameter. The filtered result must be shareable and work without client JavaScript.
- Add a server action with a deliberately invalid submission. Field errors must survive both ordinary and enhanced form submission.
- Explain every value that crosses from your server load to the client; no secret may appear in its returned object.

Primary framework documentation linked above was reviewed on 2026-10-18. Follow the adapter and API references corresponding to the installed version.
