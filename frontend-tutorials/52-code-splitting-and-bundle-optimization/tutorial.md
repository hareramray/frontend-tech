# Code splitting and bundle optimization

[Curriculum](../README.md)

Prerequisites: JavaScript modules, [performance](../50-web-performance-and-core-web-vitals/tutorial.md), and [rendering](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

## Learning outcomes

Create an on-demand JavaScript chunk, measure its loading behavior, understand tree shaking and side effects, and balance initial loading against later interaction cost. The objective is to ship useful code at the right time, with recoverable failures.

## Core concepts

Static imports establish dependencies available when a module executes. Dynamic `import()` returns a promise and provides a natural split point for a bundler. Route boundaries and rarely used tools are common candidates. Splitting a tiny frequently used helper can add overhead without meaningful savings.

Tree shaking removes unused code that the toolchain can prove unnecessary. ES module structure helps analysis, but module side effects constrain removal. Incorrectly marking a package as side-effect-free can discard required CSS or initialization. Minification, compression, splitting, and tree shaking solve different problems; none alone guarantees responsive execution. [Vite features](https://vite.dev/guide/features.html)

## Worked tutorial: load a report only when requested

1. Create a vanilla Vite project:

```sh
npm create vite@latest splitting-lab -- --template vanilla
cd splitting-lab
npm install
```

Use a supported Node.js version matching the tool's requirements. Replace `index.html`:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Deferred report</title>
<main>
  <h1>Learning report</h1>
  <button id="open">Generate report</button>
  <p id="status" role="status"></p>
  <div id="report"></div>
</main>
<script type="module" src="/src/main.js"></script>
</html>
```

2. Replace `src/main.js`:

```js
const button = document.querySelector('#open');
const status = document.querySelector('#status');
const host = document.querySelector('#report');

button.addEventListener('click', async () => {
  button.disabled = true;
  status.textContent = 'Preparing report…';
  try {
    const { renderReport } = await import('./report.js');
    renderReport(host, [12, 25, 18]);
    status.textContent = 'Report ready.';
  } catch {
    status.textContent = 'Report could not load. Check your connection and retry.';
  } finally {
    button.disabled = false;
  }
});
```

3. Create `src/report.js`:

```js
export function renderReport(host, minutes) {
  const total = minutes.reduce((sum, value) => sum + value, 0);
  const heading = document.createElement('h2');
  heading.textContent = 'Study summary';
  const paragraph = document.createElement('p');
  paragraph.textContent = `${minutes.length} sessions, ${total} minutes total.`;
  host.replaceChildren(heading, paragraph);
}
```

4. Run `npm run build`, then `npm run preview`. Open Network, filter to JavaScript, disable the browser cache for the experiment, and reload. The report module should be requested when the button is clicked, rather than needed for initial execution. The output should read “3 sessions, 55 minutes total.”
5. Inspect the generated files in `dist/assets`. Development-server requests do not represent the final production bundling strategy. This example's report is deliberately tiny, so its split demonstrates mechanics rather than an expected real-world speedup.

## Choosing boundaries and measuring value

Start with a bundle analyzer or build report and connect large chunks to user journeys. Investigate duplicated libraries, whole-library imports, locale data, icon collections, polyfills, source assets, and accidental server dependencies in client code. A dependency's installed size is not the same as its shipped size. Compare compressed transfer, parsing, evaluation, and interaction latency separately.

Prefer a split around an optional feature with substantial code, such as an editor, charting tool, or export engine. Shared dependencies can move into another chunk; opening one feature may still require several downloads. Excessive fragmentation adds request coordination and cache-management complexity.

Prefetching can hide future latency when user intent is strong, such as hovering a navigation link, but wastes bandwidth when prediction is poor. Preloading everything defeats intentional deferral. Respect constrained connections where appropriate and measure the effect on both the first route and the destination route.

## Deployment and failure handling

Hashed filenames allow immutable caching for built assets. HTML usually needs a different freshness policy so it references the correct release. A user may keep an old tab open while a new release removes the old chunks. Preserve previous assets for an appropriate overlap period or provide a recovery flow for version mismatch. An indiscriminate reload loop is not a recovery strategy.

Dynamic import failures can involve offline conditions, a missing asset, an unsupported syntax target, or a deployment inconsistency. Inspect the failed request and response content. Returning the HTML application shell for a missing JavaScript file often produces a confusing module parsing error.

## Exercises and acceptance criteria

- Add a second optional feature. Explain which dependencies are shared and confirm the observed requests match the design.
- Compare an eager import and a deferred import of a substantial real dependency. Record transferred bytes and first-use delay.
- Simulate a failed chunk request. The interface must retain an actionable error message and avoid a permanently disabled control.
- Write cache rules for HTML and hashed assets. Explain how an old open tab remains usable after deployment.

Primary references: [Vite features](https://vite.dev/guide/features.html), [Vite production builds](https://vite.dev/guide/build), and [MDN dynamic import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import). Toolchain guidance reviewed on 2026-10-18.
