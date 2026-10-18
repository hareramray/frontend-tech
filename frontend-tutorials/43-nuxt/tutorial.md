# Nuxt: Vue pages, server APIs, and universal data

[Curriculum](../README.md)

Prerequisites: [Vue single-file components, refs, and computed values](../28-vue/tutorial.md), [TypeScript basics](../20-typescript/tutorial.md), and [rendering strategies](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

## Learning outcomes

Create a file-routed Vue application, load server data without a hydration duplicate, separate server configuration from public configuration, and choose a suitable deployment target. The file layout below uses the Nuxt 4 application directory convention; choose the corresponding starter when creating this lab.

## Core and advanced concepts

Nuxt supplies routing, server rendering, data composables, configuration, and server endpoints around Vue. Application files belong under `app/`, while server handlers belong under `server/`. An application shell with `<NuxtPage />` displays the current page. Layouts support shared page structure; middleware can guide navigation, but server endpoints must enforce authorization themselves. [Nuxt application entry point](https://learn.nuxt.com/en/concepts/app-vue)

`useFetch` and `useAsyncData` integrate loading with rendering and hydration. Their returned values expose reactive data and status. They can transfer server-loaded results to the client so the initial browser render does not need an identical second fetch. `$fetch` is useful for deliberate actions, such as a button-triggered mutation. Calling it directly in a page's initial setup without the data composables can produce an unnecessary server/client duplicate. [Nuxt data fetching](https://nuxt.com/docs/4.x/getting-started/data-fetching)

## Worked tutorial: a server-fed catalog

1. Use a Node.js release supported by the selected Nuxt starter. Create the project:

```sh
npm create nuxt@latest catalog-lab -- -t v4
cd catalog-lab
npm install
```

The installer may offer package-manager and module choices; choose npm and keep optional modules minimal. Preserve its generated package manifest and lockfile. [Official Nuxt starters](https://github.com/nuxt/starter)

2. Create `server/api/topics.get.ts`:

```ts
export default defineEventHandler(() => [
  { id: 'layout', title: 'Layout', minutes: 25 },
  { id: 'forms', title: 'Forms', minutes: 40 },
  { id: 'network', title: 'Networking', minutes: 35 },
]);
```

3. Replace `app/app.vue`:

```vue
<template>
  <NuxtPage />
</template>
```

4. Create `app/pages/index.vue`:

```vue
<script setup lang="ts">
const query = ref('');
const { data: topics, status, error, refresh } = await useFetch('/api/topics');
const filtered = computed(() =>
  (topics.value ?? []).filter(topic =>
    topic.title.toLowerCase().includes(query.value.trim().toLowerCase())
  )
);
useSeoMeta({
  title: 'Study catalog',
  description: 'Find a short frontend lesson.',
});
</script>

<template>
  <main>
    <h1>Study catalog</h1>
    <label>Filter topics <input v-model="query" type="search"></label>
    <p v-if="status === 'pending'" role="status">Loading topics…</p>
    <div v-else-if="error">
      <p role="alert">Topics could not be loaded.</p>
      <button type="button" @click="refresh()">Retry</button>
    </div>
    <ul v-else-if="filtered.length">
      <li v-for="topic in filtered" :key="topic.id">
        {{ topic.title }}: {{ topic.minutes }} minutes
      </li>
    </ul>
    <p v-else>No topics match your search.</p>
  </main>
</template>
```

5. Run `npm run dev` and visit the printed local address. Typing `form` should show only Forms. Opening `/api/topics` should return JSON. Reload the home page and inspect its document response for the topic names. The page's server-rendered content and the client filter should agree.
6. Run `npm run build`, then the generated preview script, commonly `npm run preview`. Inspect `package.json` for the exact available script. Production deployment may use a different Nitro preset and start command.

## State, deployment, and security

State created at module scope can survive across requests in a server process. Never place per-user mutable data there. Use request-scoped server logic and Nuxt's SSR-aware state mechanisms where shared reactive state is needed. A key used for cached data should represent the inputs that determine its value; unrelated requests must not collide.

Private runtime configuration belongs on the server. Values under public runtime configuration are intentionally exposed to the browser. Validate external input in each server handler, and only return fields that the client is permitted to see. Route middleware cannot protect direct calls to `/api/topics` or other endpoints.

Route rules and prerendering allow different rendering and caching behavior across a project. Public documentation may be generated, while a signed-in account page requires request-specific handling. A static hosting target cannot run arbitrary server API handlers; confirm the output and runtime requirements before selecting hosting.

## Pitfalls and debugging

Hydration errors can follow browser-only APIs, time-dependent initial values, or inconsistent locale formatting. Use `onMounted` for appropriate browser work, and isolate genuinely client-only widgets deliberately. Do not wrap the whole application in a client-only boundary to hide a narrow mismatch.

When data appears stale, inspect the fetch key, reactive arguments, refresh trigger, and response caching separately. When server rendering fails, check the terminal as well as the browser console. Auto-imports are convenient, but explicit module boundaries and typed data contracts remain valuable as the application grows.

## Exercises and acceptance criteria

- Add a dynamic detail page. Both direct navigation and client navigation must load the requested topic.
- Add a simulated server error. The user must see a retry control and must not see raw stack traces.
- Move a secret API credential into private runtime configuration. Verify it does not appear in HTML, payloads, or client bundles.
- Describe which routes could be prerendered and which require a server, including a freshness policy for each.

Primary references: [Nuxt useFetch](https://nuxt.com/docs/4.x/api/composables/use-fetch), the data-fetching guide, the application-entry tutorial, and official starter repository above. Version-sensitive source review: 2026-10-18.
