# Coverage map and depth

[Curriculum](README.md)

The 72 numbered tutorials define this library's scope. Each has a worked lesson and practice tasks. This map locates related techniques so they do not disappear behind broad chapter names. Some specialized topics receive design guidance rather than a complete implementation; a single Markdown library cannot replace every specification, framework manual, or deployment provider's instructions.

## Core platform

| Technique family | Where to study | Coverage |
|---|---|---|
| URLs, origins, DNS, TLS, HTTP, browser rendering | [01](01-web-and-browser-fundamentals/tutorial.md), [34](34-http-fetch-and-rest/tutorial.md) | Request lifecycle, browser model, protocol behavior, fetch practice |
| Editors, terminals, Git, local servers | [02](02-development-environment-cli-and-git/tutorial.md), [Setup](SETUP.md) | Practical workflow and command conventions |
| Semantic content and forms | [03](03-semantic-html/tutorial.md), [04](04-html-forms-and-validation/tutorial.md), [38](38-forms-in-applications/tutorial.md) | Structure, native controls, validation, application forms |
| Accessibility and inclusive interaction | [05](05-web-accessibility/tutorial.md), [49](49-accessibility-testing/tutorial.md), [57](57-complex-accessible-interface-patterns/tutorial.md) | Foundations, manual/automated verification, complex patterns |
| CSS layout, sizing, cascade, responsive behavior | [06](06-css-foundations/tutorial.md)–[12](12-advanced-css-and-css-architecture/tutorial.md) | Worked examples and architecture tradeoffs |
| Sass, PostCSS, CSS Modules, utility CSS, CSS-in-JS | [12](12-advanced-css-and-css-architecture/tutorial.md), [24](24-build-tools-and-bundlers/tutorial.md), [26](26-components-and-design-systems/tutorial.md) | Selection and integration tradeoffs; not a separate course for each package |
| JavaScript syntax, scope, objects, collections | [13](13-javascript-fundamentals/tutorial.md)–[16](16-arrays-collections-and-immutability/tutorial.md) | Runtime concepts and practical transformations |
| DOM, event propagation, delegation, async races | [17](17-dom-events-and-interaction/tutorial.md)–[19](19-modules-errors-and-debugging/tutorial.md) | Worked browser behavior and debugging |
| TypeScript and data-boundary validation | [20](20-typescript/tutorial.md), [71](71-backend-for-frontend-and-api-contracts/tutorial.md) | Compile-time modeling versus runtime checks |
| Complexity, data structures, functional patterns | [21](21-data-structures-and-algorithms/tutorial.md), [22](22-functional-and-reactive-patterns/tutorial.md) | Frontend applications of algorithms and state transformations |

## Application development

| Technique family | Where to study | Coverage |
|---|---|---|
| Dependency management, build tools, code quality | [23](23-packages-and-dependency-management/tutorial.md)–[25](25-code-quality-and-documentation/tutorial.md) | Manifests, locks, transforms, checks, documentation |
| Design tokens and component APIs | [26](26-components-and-design-systems/tutorial.md), [56](56-ux-and-interaction-design/tutorial.md) | Reusable behavior, variants, interaction decisions |
| React, Vue, Angular, Svelte | [27](27-react/tutorial.md)–[30](30-svelte/tutorial.md) | A dedicated introductory-to-intermediate tutorial for each |
| Custom elements and Shadow DOM | [31](31-web-components/tutorial.md) | Interoperability, lifecycle, encapsulation |
| Routing, state, reducers, URL state | [32](32-routing-and-navigation/tutorial.md), [33](33-client-state-management/tutorial.md) | Ownership, navigation, state modeling |
| REST, GraphQL, caching, optimistic changes | [34](34-http-fetch-and-rest/tutorial.md)–[36](36-graphql/tutorial.md) | Contracts, errors, freshness, invalidation, mutations |
| WebSockets, SSE, reconnects | [37](37-realtime-websockets-and-sse/tutorial.md) | Live-event transport and recovery |
| Authentication, authorization, XSS, CSRF, CSP | [39](39-authentication-and-authorization/tutorial.md), [40](40-frontend-security-and-privacy/tutorial.md) | Trust boundaries, secure workflows, threat-aware implementation |
| CSR, SSR, SSG, hydration, streaming, islands | [41](41-rendering-csr-ssr-ssg-and-islands/tutorial.md) | Rendering tradeoffs and application boundaries |
| Next.js, Nuxt, SvelteKit, Astro | [42](42-nextjs/tutorial.md)–[45](45-astro/tutorial.md) | Dedicated framework workflows and worked examples |
| Other framework ecosystems and legacy code | [60](60-frontend-architecture-and-monorepos/tutorial.md), [70](70-compatibility-and-progressive-enhancement/tutorial.md), [72](72-engineering-workflow-and-maintenance/tutorial.md) | Transferable migration and maintenance principles; no claim of exhaustive package-specific coverage |

## Quality, delivery, and specialization

| Technique family | Where to study | Coverage |
|---|---|---|
| Unit, component, integration, E2E testing | [46](46-unit-testing/tutorial.md)–[48](48-end-to-end-testing/tutorial.md) | Executable testing patterns, realistic boundaries, debugging |
| Accessibility audits and assistive technology | [49](49-accessibility-testing/tutorial.md) | Automated checks plus manual workflows |
| Core Web Vitals, traces, memory, bundles | [50](50-web-performance-and-core-web-vitals/tutorial.md)–[52](52-code-splitting-and-bundle-optimization/tutorial.md) | Measurement, causes, optimization, lifecycle cleanup |
| Service workers, PWA installation, offline cache | [53](53-progressive-web-apps-and-offline/tutorial.md), [Offline project](projects/03-offline-notes/tutorial.md) | Shell caching, updates, user expectations |
| Pluralization, locale formatting, RTL, time zones | [54](54-internationalization-and-localization/tutorial.md) | Message and layout correctness across locales |
| Metadata, canonical URLs, structured data | [55](55-seo-and-structured-data/tutorial.md) | Discoverability and content representation |
| UX states, menus, dialogs, tabs, focus | [56](56-ux-and-interaction-design/tutorial.md), [57](57-complex-accessible-interface-patterns/tutorial.md) | Usability and accessible interaction contracts |
| Analytics, errors, flags, privacy | [58](58-observability-analytics-and-feature-flags/tutorial.md) | Deliberate instrumentation and staged release |
| Hosting, CDNs, CI/CD, environments, rollback | [59](59-deployment-ci-cd-and-environments/tutorial.md) | Reproducible delivery and operational validation |
| Monorepos, package boundaries, microfrontends | [60](60-frontend-architecture-and-monorepos/tutorial.md) | Runnable workspace and architecture decisions |
| IndexedDB, storage quotas, migration, sync | [61](61-storage-indexeddb-and-data-lifecycle/tutorial.md) | Transactions and lifecycle; sync protocol design rather than a hosted sync product |
| Permissions, observers, clipboard, devices | [62](62-browser-apis-and-permissions/tutorial.md) | Runnable camera lifecycle and capability-selection guidance |
| Workers, transferables, shared memory, Wasm | [63](63-workers-webassembly-and-concurrency/tutorial.md) | Runnable worker; advanced execution tradeoffs |
| Canvas, SVG, WebGL, WebGPU, shaders | [64](64-canvas-webgl-and-webgpu/tutorial.md) | Runnable 2D graphics and GPU pipeline foundations |
| Audio graphs, captions, codecs, streaming | [65](65-audio-video-and-streaming/tutorial.md) | Runnable tone and media-delivery architecture |
| WebRTC, ICE, STUN/TURN, SFU/MCU | [66](66-webrtc-and-peer-communication/tutorial.md) | Runnable local data channel and production connectivity design |
| Touch, gestures, web/hybrid/native choices | [67](67-mobile-touch-and-cross-platform/tutorial.md) | Pointer interaction and delivery tradeoffs |
| Files, resumable uploads, rich text, drag/drop | [68](68-file-uploads-drag-drop-and-editors/tutorial.md) | Runnable file preview and robust workflow design |
| Search, sorting, tables, charts, virtualization | [69](69-search-tables-and-data-visualization/tutorial.md) | Runnable table and scalable data-interface decisions |
| Feature detection, fallbacks, browser support | [70](70-compatibility-and-progressive-enhancement/tutorial.md) | Runnable enhancement and compatibility policy |
| BFF, OpenAPI, adapters, schema validation | [71](71-backend-for-frontend-and-api-contracts/tutorial.md) | Runnable server adaptation and contract evolution |
| Reviews, incidents, upgrades, technical debt | [72](72-engineering-workflow-and-maintenance/tutorial.md) | Regression-driven repair and maintenance workflow |

## How to extend the library

When adding a topic, define its prerequisites, learning outcomes, runnable environment, core model, worked tutorial, failure modes, measurable exercises, and primary references. Add it to the index and this map. Prefer a new focused chapter when the subject has a distinct learning goal; use a section when it is a variation of an existing technique.

Backend internals, database administration, native platform internals, advanced shader mathematics, compiler construction, and full security auditing extend beyond a frontend curriculum. Their frontend-facing boundaries are covered here; mastering those fields requires their own dedicated study paths.
