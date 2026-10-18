# Frontend glossary

[Curriculum](README.md)

| Term | Meaning |
|---|---|
| Accessibility tree | Browser representation of interface semantics exposed to assistive technology. |
| Accessible name | The name assistive technology uses to identify an element, derived from its content and labeling rules. |
| Adapter | Code that translates one interface or data shape into another at a boundary. |
| ARIA | Attributes and roles that describe accessibility semantics when native HTML alone is insufficient. |
| Artifact | A produced build output or package that can be tested, stored, and deployed. |
| Atomic change | A change applied as one indivisible unit in the relevant system; context determines the guarantee. |
| Authentication | Establishing an identity or authenticated session. |
| Authorization | Deciding whether an identity may perform an operation on a resource. |
| Backpressure | A mechanism that prevents producers from overwhelming slower consumers. |
| BFF | Backend for frontend: a server layer tailored to a particular client experience. |
| Box model | CSS content, padding, border, and margin dimensions around an element. |
| Bundler | Tool that resolves and transforms modules and assets into deployable outputs. |
| Cache invalidation | Marking or replacing cached data when it should no longer satisfy a request. |
| Cascade | Rules that choose winning CSS declarations across origins, importance, layers, specificity, and ordering. |
| CDN | Content delivery network that serves resources through distributed infrastructure. |
| CI/CD | Automated integration checks and delivery/deployment workflows; the exact deployment policy varies. |
| Closure | A function's ability to retain access to its lexical environment. |
| CLS | Cumulative Layout Shift, a measure of unexpected visual layout movement. |
| Code splitting | Dividing application code into separately loadable pieces. |
| Component | A reusable unit of interface structure, behavior, and an explicit API. |
| Composition | Building behavior by combining smaller units rather than relying on a deep inheritance hierarchy. |
| Contract | Agreed expectations for inputs, outputs, failures, and behavior across a boundary. |
| CORS | Browser-enforced rules governing whether scripts can access certain cross-origin responses. |
| CSP | Content Security Policy, a browser policy restricting selected resource and execution behaviors. |
| CSR | Client-side rendering: generating interface content primarily in the browser. |
| CSRF | Cross-site request forgery: inducing an authenticated browser to perform an unwanted request. |
| Custom element | A browser-registered HTML element with application-defined behavior. |
| Debounce | Delay work until activity has paused for a chosen interval. |
| Declarative UI | Describing the desired interface for state while a rendering system applies updates. |
| Dependency injection | Supplying dependencies from outside a unit rather than constructing all of them internally. |
| Derived state | A value calculated from existing state rather than stored as another independent source of truth. |
| Design token | A named design decision such as a semantic color, spacing value, or typography choice. |
| DOM | Document Object Model: the browser's programmable tree representation of a document. |
| E2E test | An end-to-end test that exercises a complete workflow through a deployed or test application. |
| Effect | Work that synchronizes with an external system or produces behavior beyond pure calculation. |
| ESM | ECMAScript modules, using `import` and `export` with language-defined module semantics. |
| Event delegation | Handling descendant events through a suitable ancestor listener. |
| Event loop | Scheduling model that coordinates tasks, microtasks, and rendering opportunities. |
| Feature detection | Checking for a capability instead of inferring it solely from browser identity. |
| Feature flag | Runtime or build configuration controlling access to a feature or variant. |
| Focus | The current target of keyboard input and a key part of accessible interaction. |
| Garbage collection | Runtime reclamation of memory that is no longer reachable according to its memory model. |
| GraphQL | A typed query language and execution model for requesting data from an API schema. |
| Hydration | Connecting browser-side behavior to server-generated or pre-rendered interface output. |
| ICE | Interactive Connectivity Establishment, used to find a usable WebRTC connection path. |
| Idempotency | Repeating an operation has the same intended effect as performing it once, under its contract. |
| Immutability | Treating a value as unchangeable and creating a new value to represent an update. |
| IndexedDB | Browser database API for structured data, indexes, and transactions. |
| INP | Interaction to Next Paint, a field metric of interaction responsiveness. |
| Internationalization | Designing software so content and behavior can be adapted across languages and locales. |
| Island | An independently interactive region in a page that can hydrate separately from other content. |
| JSX | Syntax that describes elements within JavaScript in supported tooling; it is transformed before execution. |
| LCP | Largest Contentful Paint, a metric concerned with the rendering time of major visible content. |
| Lexical scope | Name resolution determined by where code is written. |
| Live region | Accessibility mechanism for announcing relevant content changes without moving focus. |
| Localization | Adapting content, formatting, and conventions for a particular locale. |
| Lockfile | Recorded dependency resolution used to make installations more repeatable. |
| Memoization | Reusing a computed result for matching inputs under a defined equality strategy. |
| Microfrontend | An independently owned or delivered frontend portion composed into a larger experience. |
| Microtask | Scheduled work processed at defined checkpoints, including promise reactions. |
| Monorepo | One repository containing multiple projects or packages. |
| Mutation | A state change; in APIs, often an operation that changes server data. |
| Optimistic update | Showing an anticipated mutation result before server confirmation, with a failure strategy. |
| Origin | Scheme, host, and port tuple used for important browser security boundaries. |
| Polyfill | Code that supplies selected missing runtime behavior in an environment. |
| Progressive enhancement | Building useful baseline behavior and adding capabilities when available. |
| Promise | Object representing the eventual fulfillment or rejection of an asynchronous operation. |
| PWA | Progressive web application using selected web capabilities such as installation or offline behavior. |
| Race condition | Outcome that depends on timing or ordering not adequately controlled by the program. |
| Reducer | A function that computes next state from previous state and an action. |
| Reflow/layout | Browser calculation of element geometry and placement. |
| Rendering | Producing visible or structural interface output; the term also refers to browser painting pipelines. |
| REST | Architectural style for networked systems; many HTTP APIs use only some of its constraints. |
| Roving tabindex | Composite-widget pattern where one item is tabbable and arrow keys move internal focus. |
| Runtime validation | Checking actual values during execution instead of relying only on static types. |
| Semantic HTML | Elements chosen for their meaning and behavior rather than only their appearance. |
| Service worker | Event-driven worker that can intercept eligible requests and support selected background capabilities. |
| Shadow DOM | Encapsulated DOM subtree with particular styling and event-boundary behavior. |
| Signal | Reactive value abstraction that can notify dependents when it changes; framework semantics vary. |
| Source map | Mapping from generated code to its source for debugging. |
| Specificity | Selector weight used at a particular stage of CSS cascade resolution. |
| SSE | Server-Sent Events: a server-to-client event stream commonly consumed through EventSource. |
| SSG | Static site generation: producing page output ahead of requests. |
| SSR | Server-side rendering: producing interface output on a server for a request or server workflow. |
| STUN | Protocol used in connectivity discovery, including WebRTC network traversal. |
| Structured clone | Browser/JavaScript mechanism for copying supported structured values across certain boundaries. |
| Throttle | Limit work to a maximum frequency or rate. |
| Transaction | Group of database operations with defined atomicity and lifecycle rules. |
| Transferable | Resource whose ownership can move across contexts without copying its underlying data. |
| Tree shaking | Build optimization that removes unused code when analysis and module semantics permit it. |
| TURN | Relay protocol used when a direct peer connectivity path is unavailable or unsuitable. |
| Type narrowing | Refining a broad static type through checks and control flow. |
| Virtualization | Rendering a subset of a large collection based on the visible region. |
| WebAssembly | Portable binary compilation target executable through supported runtimes. |
| WebSocket | Persistent bidirectional communication protocol available through browser and server APIs. |
| Worker | Separate execution context used for suitable background computation or platform-specific tasks. |
| XSS | Cross-site scripting: attacker-controlled content executing in a trusted application's browser context. |

Terminology can differ between frameworks. When two tools use the same word, inspect their lifecycle and guarantees rather than assuming the implementations are interchangeable.
