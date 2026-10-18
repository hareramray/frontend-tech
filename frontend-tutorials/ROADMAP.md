# Learning roadmap

[Curriculum](README.md) · [Projects](projects/README.md)

Study by demonstrated competence rather than a fixed number of days. A chapter is complete when you can explain the core idea, reproduce its example, diagnose a deliberate failure, and meet the exercise criteria. Reading every framework before building anything is unnecessary.

## Stage 1: build understandable pages

Read chapters 01–12. Learn the browser request/rendering model, a local development workflow, semantic HTML, forms, accessibility, CSS, layout, responsive behavior, assets, and motion. Keep DevTools open and inspect the DOM, computed styles, layout, and network requests.

Build the [accessible portfolio](projects/01-accessible-portfolio/tutorial.md). Begin with content and navigation, then add layout and visual design. Evaluate keyboard behavior before adding animation. Practice debugging an overflowing grid item, an unlabeled form field, a missing image, and a specificity conflict.

**Milestone:** you can create a multi-page site that remains usable at a narrow viewport and increased zoom, has clear headings and focus states, and does not depend on JavaScript for basic navigation.

## Stage 2: understand browser programming

Read chapters 13–22. Prioritize values and identity, scope, closures, collection transformations, DOM events, async behavior, errors, and modules before reaching for an application framework. Add TypeScript after you can recognize the runtime behavior it describes.

Build small interactions: a validated form, searchable list, cancellable request, and state-driven panel. Predict console output for tasks and microtasks. Practice diagnosing stale closures, accidental mutation, missing event cleanup, and an out-of-order response.

**Milestone:** you can explain why a function runs, what state it reads, who owns that state, what happens if a request fails, and how resources are cleaned up.

## Stage 3: use tools and one component model

Read chapters 23–26, then choose one of React, Vue, Angular, or Svelte in chapters 27–30. Read chapter 31 to understand interoperable custom elements even if you do not adopt them. Learn routing, state, fetch, caching, and forms in chapters 32–38. GraphQL and real-time transport can follow when your project needs them.

Build the [API dashboard](projects/02-api-dashboard/tutorial.md). Use source data, view state, and derived state deliberately. Make empty, loading, error, and success states equally visible. Store shareable filters in the URL and test back navigation.

**Milestone:** you can build a feature without duplicating derived state, leaking effects, mutating shared data, or letting an obsolete request replace current results.

## Stage 4: respect server and security boundaries

Read chapters 39–41 and the rendering framework relevant to your chosen component framework: Next.js, Nuxt, SvelteKit, or Astro in chapters 42–45. Add chapter 71 for API contracts and a BFF when useful.

Explain which code runs in the browser, on the server, or during a build. Identify where authentication is established and where authorization is enforced. Inspect cookies, cache headers, client bundles, and serialized data. Practice a direct API request that bypasses the UI to prove that server access checks exist.

**Milestone:** you can draw the request and data path, identify trust boundaries, prevent private data from entering a shared cache, and explain why client-side validation is insufficient.

## Stage 5: verify the user experience

Read chapters 46–58. Add unit tests for pure behavior, component tests for interactions, and a small number of end-to-end tests for critical journeys. Combine automated accessibility checks with keyboard and screen-reader testing. Measure performance before optimizing.

Add internationalization, SEO, offline behavior, or complex widgets according to project needs. Learn their constraints even when you defer implementation. A dialog, a translated date, and an offline save indicator each require more than visual styling.

**Milestone:** you can produce evidence that a feature works, identify what the evidence does not cover, and prioritize a real user-impacting defect over an arbitrary score.

## Stage 6: ship and maintain

Read chapters 59–60 and 72. Build the [production capstone](projects/04-production-capstone/tutorial.md). Create reproducible CI, a preview environment, deliberate caching, release identification, observability, and a rollback process. Review dependency updates and document key decisions.

**Milestone:** a clean checkout can pass checks and build; deployed deep links work; the release can be identified; private configuration stays server-side; and a failed release can be recovered without improvisation.

## Stage 7: specialize

Choose advanced tracks after the common foundation:

| Track | Chapters | Practical proof |
|---|---|---|
| Local-first applications | 53, 61, 63, 68 | [Offline notes](projects/03-offline-notes/tutorial.md) with honest persistence and recovery |
| Interactive graphics | 10, 11, 50, 51, 63, 64, 69 | Accessible chart or editor with measured rendering and cleanup |
| Real-time collaboration | 18, 35–37, 61, 66, 68 | Reconnect-safe communication with explicit conflict semantics |
| Media applications | 62, 65–67 | Permission-aware player or calling lab with captions and resource cleanup |
| Design systems | 05–12, 26, 31, 47, 49, 54, 57 | Documented components with states, tokens, keyboard behavior, and localization |
| Frontend platform | 23–25, 40, 48, 52, 58–60, 70–72 | Reproducible packages and releases with enforceable boundaries |

## A repeatable study session

1. Read the concepts and write a prediction about the example.
2. Run the example in a separate practice folder.
3. Inspect its DOM, state, network, or output as appropriate.
4. Change one assumption: missing data, slow response, narrow viewport, denied permission, or invalid input.
5. Complete the exercise and record evidence against its acceptance criteria.
6. Write a short explanation without copying the tutorial.

Use a progress log with columns for topic, example run, exercise evidence, unresolved questions, and next review. Revisit a topic when a project exposes a gap. Do not confuse recognizing terminology with being able to build and debug the behavior.
