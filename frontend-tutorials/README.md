# Frontend curriculum

[Repository home](../README.md) · [Setup](SETUP.md) · [Roadmap](ROADMAP.md) · [Coverage](COVERAGE.md) · [Projects](projects/README.md)

This library teaches the major skills needed to build, ship, and maintain browser interfaces. Follow the sequence for foundations, then select a framework and project. Each numbered link opens a dedicated folder containing its tutorial. The descriptions below define the topic coverage; they are not claims that one chapter replaces an entire language specification.

Use the examples actively: predict the result, run them in a practice folder, inspect behavior in DevTools, change an input, and complete the acceptance criteria. Code labeled as a fragment needs the surrounding application described in its chapter. Consult [setup](SETUP.md) before running anything.

## Web, HTML, accessibility, and CSS

| # | Tutorial | Main techniques |
|---|---|---|
| 01 | [Web and browser fundamentals](01-web-and-browser-fundamentals/tutorial.md) | URLs, DNS, TLS, HTTP, origins, browser rendering, DevTools |
| 02 | [Development environment, CLI, and Git](02-development-environment-cli-and-git/tutorial.md) | Editors, terminals, local servers, version control, branches, commits |
| 03 | [Semantic HTML](03-semantic-html/tutorial.md) | Document structure, landmarks, headings, links, lists, tables, metadata |
| 04 | [HTML forms and validation](04-html-forms-and-validation/tutorial.md) | Labels, input types, constraints, errors, submission, server validation |
| 05 | [Web accessibility](05-web-accessibility/tutorial.md) | Semantics, keyboard access, focus, names, ARIA, contrast, inclusive content |
| 06 | [CSS foundations](06-css-foundations/tutorial.md) | Cascade, specificity, inheritance, box model, selectors, units |
| 07 | [Flexbox and Grid](07-flexbox-and-grid/tutorial.md) | One- and two-dimensional layout, alignment, tracks, intrinsic sizing |
| 08 | [Responsive design](08-responsive-design/tutorial.md) | Flexible layouts, breakpoints, media/container queries, zoom, content flow |
| 09 | [Typography, color, and visual design](09-typography-color-and-visual-design/tutorial.md) | Type scales, readable measures, color systems, hierarchy, font loading |
| 10 | [Images, SVG, and media assets](10-images-svg-and-media-assets/tutorial.md) | Responsive images, formats, SVG, dimensions, alternative text, optimization |
| 11 | [Animation and motion](11-animation-and-motion/tutorial.md) | Transitions, keyframes, timing, rendering cost, reduced motion |
| 12 | [Advanced CSS and architecture](12-advanced-css-and-css-architecture/tutorial.md) | Layers, tokens, component boundaries, modern selectors, maintainable styling |

## JavaScript, TypeScript, and tooling

| # | Tutorial | Main techniques |
|---|---|---|
| 13 | [JavaScript fundamentals](13-javascript-fundamentals/tutorial.md) | Values, variables, coercion, operators, conditions, loops |
| 14 | [Functions, scope, and closures](14-functions-scope-and-closures/tutorial.md) | Lexical scope, callbacks, higher-order functions, closures, `this` |
| 15 | [Objects, prototypes, and classes](15-objects-prototypes-and-classes/tutorial.md) | Properties, delegation, constructors, classes, composition |
| 16 | [Arrays, collections, and immutability](16-arrays-collections-and-immutability/tutorial.md) | Array transforms, Map/Set, identity, copying, immutable updates |
| 17 | [DOM, events, and interaction](17-dom-events-and-interaction/tutorial.md) | Querying, safe updates, propagation, delegation, cleanup |
| 18 | [Promises, async, and the event loop](18-promises-async-and-the-event-loop/tutorial.md) | Tasks, microtasks, async/await, concurrency, cancellation, races |
| 19 | [Modules, errors, and debugging](19-modules-errors-and-debugging/tutorial.md) | ESM, module boundaries, exceptions, debugging, error context |
| 20 | [TypeScript](20-typescript/tutorial.md) | Unions, narrowing, interfaces, generics, utility types, runtime boundaries |
| 21 | [Data structures and algorithms](21-data-structures-and-algorithms/tutorial.md) | Complexity, indexing, lookup, sorting, trees, frontend problem solving |
| 22 | [Functional and reactive patterns](22-functional-and-reactive-patterns/tutorial.md) | Pure functions, composition, reducers, subscriptions, derived state |
| 23 | [Packages and dependency management](23-packages-and-dependency-management/tutorial.md) | npm, manifests, lockfiles, semantic versions, dependency risk |
| 24 | [Build tools and bundlers](24-build-tools-and-bundlers/tutorial.md) | Development servers, transforms, assets, production builds, configuration |
| 25 | [Code quality and documentation](25-code-quality-and-documentation/tutorial.md) | Linting, formatting, type checks, API contracts, maintainable documentation |

## Components, frameworks, and application data

| # | Tutorial | Main techniques |
|---|---|---|
| 26 | [Components and design systems](26-components-and-design-systems/tutorial.md) | Component APIs, tokens, variants, states, composition, documentation |
| 27 | [React](27-react/tutorial.md) | JSX, props, state, hooks, effects, identity, component composition |
| 28 | [Vue](28-vue/tutorial.md) | Templates, Composition API, refs, computed values, events, lifecycle |
| 29 | [Angular](29-angular/tutorial.md) | Components, templates, dependency injection, signals, application structure |
| 30 | [Svelte](30-svelte/tutorial.md) | Components, runes, derived state, events, reactivity, lifecycle |
| 31 | [Web Components](31-web-components/tutorial.md) | Custom elements, attributes, Shadow DOM, lifecycle, interoperability |
| 32 | [Routing and navigation](32-routing-and-navigation/tutorial.md) | URL state, history, routes, navigation behavior, focus, deep links |
| 33 | [Client state management](33-client-state-management/tutorial.md) | Local/shared state, reducers, stores, derived state, ownership |
| 34 | [HTTP, fetch, and REST](34-http-fetch-and-rest/tutorial.md) | Methods, status codes, JSON, aborting, CORS, retries, error states |
| 35 | [Server state and caching](35-server-state-and-caching/tutorial.md) | Query keys, freshness, invalidation, optimistic updates, request deduplication |
| 36 | [GraphQL](36-graphql/tutorial.md) | Schemas, queries, variables, mutations, errors, pagination, client behavior |
| 37 | [WebSockets and SSE](37-realtime-websockets-and-sse/tutorial.md) | Live events, reconnects, ordering, resynchronization, transport choice |
| 38 | [Forms in applications](38-forms-in-applications/tutorial.md) | Controlled state, field errors, async submission, validation, complex forms |

## Security and rendering frameworks

| # | Tutorial | Main techniques |
|---|---|---|
| 39 | [Authentication and authorization](39-authentication-and-authorization/tutorial.md) | Sessions, cookies, identity flows, access checks, login/logout behavior |
| 40 | [Frontend security and privacy](40-frontend-security-and-privacy/tutorial.md) | XSS, CSRF, CSP, untrusted data, dependencies, privacy boundaries |
| 41 | [CSR, SSR, SSG, and islands](41-rendering-csr-ssr-ssg-and-islands/tutorial.md) | Rendering location, hydration, streaming, static output, cache boundaries |
| 42 | [Next.js](42-nextjs/tutorial.md) | App Router, server/client boundaries, data, rendering, mutations |
| 43 | [Nuxt](43-nuxt/tutorial.md) | Vue application structure, routes, server data, hydration, runtime config |
| 44 | [SvelteKit](44-sveltekit/tutorial.md) | Routes, load functions, server actions, progressive forms, adapters |
| 45 | [Astro](45-astro/tutorial.md) | Content-oriented pages, components, islands, selective hydration |

## Testing, performance, and product quality

| # | Tutorial | Main techniques |
|---|---|---|
| 46 | [Unit testing](46-unit-testing/tutorial.md) | Behavioral tests, boundaries, fixtures, deterministic assertions |
| 47 | [Component and integration testing](47-component-and-integration-testing/tutorial.md) | User-facing queries, interaction, network boundaries, realistic integration |
| 48 | [End-to-end testing](48-end-to-end-testing/tutorial.md) | Browser workflows, locators, isolation, traces, reliable CI tests |
| 49 | [Accessibility testing](49-accessibility-testing/tutorial.md) | Automated checks, keyboard audits, screen-reader workflows, limitations |
| 50 | [Web performance and Core Web Vitals](50-web-performance-and-core-web-vitals/tutorial.md) | LCP, INP, CLS, lab/field data, critical resources, performance budgets |
| 51 | [Profiling and memory management](51-profiling-and-memory-management/tutorial.md) | Performance traces, long tasks, allocations, leaks, lifecycle cleanup |
| 52 | [Code splitting and bundle optimization](52-code-splitting-and-bundle-optimization/tutorial.md) | Dynamic imports, tree shaking, chunking, loading policy, dependencies |
| 53 | [PWAs and offline behavior](53-progressive-web-apps-and-offline/tutorial.md) | Manifest, service workers, caches, update lifecycle, offline UX |
| 54 | [Internationalization and localization](54-internationalization-and-localization/tutorial.md) | Messages, plurals, locale formatting, RTL, time zones, translated layouts |
| 55 | [SEO and structured data](55-seo-and-structured-data/tutorial.md) | Crawlable content, titles, canonical URLs, metadata, structured data |
| 56 | [UX and interaction design](56-ux-and-interaction-design/tutorial.md) | Task flows, feedback, error recovery, usability, interface states |
| 57 | [Complex accessible interfaces](57-complex-accessible-interface-patterns/tutorial.md) | Dialogs, tabs, menus, focus management, keyboard patterns, complexity |
| 58 | [Observability, analytics, and flags](58-observability-analytics-and-feature-flags/tutorial.md) | Error reporting, events, privacy, release context, gradual delivery |

## Delivery, advanced browser work, and engineering

| # | Tutorial | Main techniques |
|---|---|---|
| 59 | [Deployment, CI/CD, and environments](59-deployment-ci-cd-and-environments/tutorial.md) | Reproducible builds, hosting, public config, caching, rollout, rollback |
| 60 | [Architecture and monorepos](60-frontend-architecture-and-monorepos/tutorial.md) | Feature boundaries, workspaces, package APIs, task graphs, microfrontends |
| 61 | [Storage, IndexedDB, and data lifecycle](61-storage-indexeddb-and-data-lifecycle/tutorial.md) | Transactions, migration, quotas, offline data, sync conflicts, account isolation |
| 62 | [Browser APIs and permissions](62-browser-apis-and-permissions/tutorial.md) | Media capture, permission states, observers, device APIs, resource cleanup |
| 63 | [Workers, WebAssembly, and concurrency](63-workers-webassembly-and-concurrency/tutorial.md) | Background computation, messages, transfers, shared memory, Wasm |
| 64 | [Canvas, WebGL, and WebGPU](64-canvas-webgl-and-webgpu/tutorial.md) | Bitmap/vector choice, resolution, shaders, GPU resources, graphics accessibility |
| 65 | [Audio, video, and streaming](65-audio-video-and-streaming/tutorial.md) | Media controls, captions, audio graphs, codecs, adaptive delivery |
| 66 | [WebRTC and peer communication](66-webrtc-and-peer-communication/tutorial.md) | Signaling, ICE, STUN/TURN, data channels, calls, multiparty tradeoffs |
| 67 | [Mobile, touch, and cross-platform](67-mobile-touch-and-cross-platform/tutorial.md) | Pointer events, gestures, mobile constraints, PWA/hybrid/native choices |
| 68 | [Uploads, drag and drop, and editors](68-file-uploads-drag-drop-and-editors/tutorial.md) | Files, preview, upload state, large transfers, editing models, accessible reordering |
| 69 | [Search, tables, and visualization](69-search-tables-and-data-visualization/tutorial.md) | Sorting, filtering, pagination, selection, virtualization, chart integrity |
| 70 | [Compatibility and progressive enhancement](70-compatibility-and-progressive-enhancement/tutorial.md) | Support policy, feature detection, fallbacks, polyfills, browser testing |
| 71 | [BFFs and API contracts](71-backend-for-frontend-and-api-contracts/tutorial.md) | Server boundaries, adaptation, runtime validation, OpenAPI, compatibility |
| 72 | [Engineering workflow and maintenance](72-engineering-workflow-and-maintenance/tutorial.md) | Reproduction, regression tests, review, upgrades, incidents, technical debt |

## Guided projects

| Project | What you build | Suggested prerequisites |
|---|---|---|
| [Accessible portfolio](projects/01-accessible-portfolio/tutorial.md) | A responsive, semantic site with a tested navigation and contact flow | 01–12, 17, 49 |
| [API dashboard](projects/02-api-dashboard/tutorial.md) | Search, sorting, URL filters, data fetching, and robust application states | 13–38 plus one framework |
| [Offline notes](projects/03-offline-notes/tutorial.md) | Device-local persistence, explicit save states, offline shell, upgrade behavior | 18, 40, 53, 61 |
| [Production capstone](projects/04-production-capstone/tutorial.md) | A complete application with contracts, tests, security, observability, and deployment | Relevant chapters across the full curriculum |

## Supporting material

- [Setup](SETUP.md): prerequisites, local servers, runnable-versus-contextual examples, and command conventions.
- [Roadmap](ROADMAP.md): ordered study paths and milestone criteria.
- [Coverage map](COVERAGE.md): where to find related techniques and how deeply each is treated.
- [Cheatsheets](CHEATSHEETS.md): compact reminders for daily work.
- [Glossary](GLOSSARY.md): vocabulary used throughout the library.
- [Troubleshooting](TROUBLESHOOTING.md): practical diagnosis by symptom.
- [Validation record](VALIDATION.md): checks performed and their limits.

Version-sensitive examples and references were prepared on 18 October 2026. APIs, framework defaults, compatibility, and recommended tooling can change. Use the linked primary documentation and your installed versions when a scaffold differs; do not replace a carefully chosen project version merely to match a screenshot or filename.
