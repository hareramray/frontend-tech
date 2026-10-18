# Troubleshooting by symptom

[Curriculum](README.md) · [Setup](SETUP.md)

Start with the smallest reproduction. Record the exact URL, command, browser, runtime, and first error. Change one variable at a time. Clearing every cache or reinstalling every dependency can hide the evidence needed to diagnose the cause.

## The page is blank

Check the terminal build output and browser Console. A JavaScript exception can prevent initialization. Inspect Network for the HTML document, entry script, chunks, and stylesheets. A missing module often produces an HTML error page; the resulting MIME-type error is a clue that the path or deployment rewrite is wrong.

Confirm that the expected root element exists, the script executes after or can access it, and the framework mounts into the correct element. Inspect whether CSS accidentally hides content. Test the production build separately from the development server.

## A module, worker, or fetch fails only from a file URL

Use a local HTTP server from the exercise directory. `file://` has different origin and loading behavior and is not a substitute for normal web hosting. Confirm relative paths against the importing module's URL, not against whichever terminal directory you happen to remember.

For workers, inspect their own errors and module setting. For WebAssembly streaming, verify the response and MIME type. For secure-context APIs, inspect the actual origin and context security.

## npm or a framework command fails

Confirm the current directory and the intended `package.json`. Check Node and package-manager requirements in the project's official documentation. On Windows, try `npm.cmd` if execution policy blocks the PowerShell wrapper. Read the first installation error before deleting files or adding force flags.

A lockfile mismatch can make `npm ci` fail intentionally. Reconcile the manifest and lockfile with an ordinary reviewed install, then commit them together. Dependency conflicts often indicate incompatible major versions; bypassing the check does not prove runtime compatibility.

## CSS does not apply or layout overflows

Inspect the element and computed style. Verify the stylesheet loaded, the selector matches, and the declaration is valid. Identify the winning rule before increasing specificity. Cascade layers can explain why a seemingly more specific selector loses.

For overflow, inspect fixed widths, long text, intrinsic image sizes, `min-width`, and grid/flex minimum sizing. Try `min-width: 0` on the relevant flex/grid child when its minimum content size is the cause. Test actual content and zoom. Avoid using `overflow: hidden` as a universal repair because it can hide controls and focus outlines.

## A request returns data but the UI still reports an error

Inspect the status, content type, body, redirects, and credentials. `fetch` does not reject merely because the server returned 404 or 500. Conversely, parsing JSON from an empty 204 response can throw after a successful operation.

Check the schema expected by the UI. A successful HTTP response with an unexpected shape is still unusable data. Validate at the boundary and log non-sensitive diagnostic context. If the response is CORS-blocked, fix the server policy or development proxy; `no-cors` gives an opaque response, not readable JSON.

## Old results replace new results

Reproduce with network throttling and two quick inputs. Track request identity, cancel obsolete work where possible, and prevent old responses from committing state. Include all relevant inputs in cache keys. An effect with incorrect dependencies can capture old values or run too often.

For autosave, distinguish the revision being saved from the current draft revision. A debounce timer does not serialize existing network or database writes. Define conflict and retry semantics instead of relying on timing luck.

## A framework renders repeatedly or hydration fails

Inspect state changes during render, effects that update their own dependencies, unstable object/function identities, and subscriptions without cleanup. Framework development diagnostics can intentionally expose lifecycle problems; removing them can conceal the bug.

Hydration requires compatible server and initial browser output. Check time-dependent values, random IDs, browser-only APIs, invalid HTML nesting, locale differences, and user-specific data. Move truly browser-only work into the correct lifecycle or component boundary rather than suppressing every mismatch.

## The app works locally but fails after deployment

Check base paths, case-sensitive filenames, environment variables, build output directory, framework adapter, and server runtime. Direct deep links may need route handling that differs from asset requests. Browser-exposed environment variables are public and may be substituted at build time.

Inspect HTML and asset cache headers. A cached HTML file can reference removed chunks. A service worker can serve older assets despite a fresh network deployment. Verify the actual release ID before debugging a different local revision.

## Offline data disappears or updates seem stuck

Inspect IndexedDB, Cache Storage, service-worker registration, scope, and lifecycle state. Cached resources and user records are different stores. A waiting service worker is not necessarily broken; an older controlled tab may still be open.

Test schema upgrades with multiple tabs. Do not clear storage containing the user's only copy of data as a first troubleshooting step. Export or preserve it when possible. Browser quotas and eviction mean local persistence requires an honest recovery and backup story.

## The interface feels slow

Measure a production build on a representative device. Separate network delay, main-thread work, rendering, and server latency. Look for oversized images, repeated data fetches, expensive list operations, synchronous storage, layout thrashing, or too much JavaScript.

A worker may improve responsiveness without reducing total elapsed time. Memoization may add overhead when the computation is cheap. Virtualization may help a large list but complicates accessibility. Choose the fix after identifying the actual bottleneck.

## A test fails intermittently

Check shared state, unawaited work, real external services, time assumptions, and selectors tied to incidental markup. Use user-visible roles and names where possible. Wait for meaningful conditions instead of arbitrary sleeps. Inspect traces and screenshots from the failed run.

Do not repeatedly increase timeouts until a race becomes harder to reproduce. Fix the isolation or synchronization issue, then keep a regression test that demonstrates the behavior.
