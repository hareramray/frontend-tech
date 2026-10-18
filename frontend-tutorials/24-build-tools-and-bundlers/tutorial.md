# 24. Build tools and bundlers

[Curriculum](../README.md)

Prerequisites: [packages](../23-packages-and-dependency-management/tutorial.md) and [modules](../19-modules-errors-and-debugging/tutorial.md).

## What you will learn

Explain the difference between development serving, transformation, bundling, and production hosting. Build a small Vite application, load an optional module on demand, and inspect the resulting production artifacts.

## Concepts and decisions

A browser understands JavaScript, CSS, HTML, and asset URLs. A source project may also contain TypeScript, JSX, framework files, package imports, or preprocessing syntax. A build pipeline resolves dependencies, transforms unsupported syntax, assembles outputs, and emits assets. Minification shortens code; compression reduces transferred bytes at the server or CDN; tree shaking removes eligible unused code. These are separate operations.

Development servers optimize feedback through fast startup and hot updates. Production builds optimize deployable output. A project working in the development server does not prove its base path, caching headers, lazy chunks, or production environment values are correct. Vite documents separate `dev`, `build`, and local `preview` workflows and supports plain HTML entry points. See [Vite's getting-started guide](https://vite.dev/guide/).

Static ESM imports expose dependency structure to analysis. Dynamic imports create asynchronous boundaries that may become separate chunks. Splitting every tiny module can increase request and scheduling overhead; split by meaningful optional features or routes. CSS and images also contribute substantially to page cost, so JavaScript size alone is an incomplete performance measure.

## Worked tutorial: load a report only when requested

Use Node.js 22.12 or newer and npm, subject to the installed Vite release's engine requirement. In a new `build-lab` directory run:

```sh
npm init -y
npm install --save-dev vite
npm pkg set type=module
npm pkg set scripts.dev="vite"
npm pkg set scripts.build="vite build"
npm pkg set scripts.preview="vite preview"
```

Create `index.html`:

```html
<!doctype html>
<html lang="en">
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Build laboratory</title>
  <button id="report" type="button">Load report</button>
  <p id="result" role="status"></p>
  <script type="module" src="/src/main.js"></script>
</html>
```

Create `src/main.js` and `src/report.js`:

```js
// src/main.js
import './style.css';
const button = document.querySelector('#report');
const result = document.querySelector('#result');
button.addEventListener('click', async () => {
  button.disabled = true;
  result.textContent = 'Loading report…';
  try {
    const { summarize } = await import('./report.js');
    result.textContent = summarize([12, 18, 30]);
  } catch (error) {
    result.textContent = 'Report unavailable. Try again.';
    console.error(error);
  } finally {
    button.disabled = false;
  }
});
```

```js
// src/report.js
export function summarize(values) {
  return `Total: ${values.reduce((sum, value) => sum + value, 0)}`;
}
```

Create `src/style.css`:

```css
body { font: 1.1rem system-ui; max-width: 45rem; margin: 3rem auto; padding: 1rem; }
button { font: inherit; padding: .7rem 1rem; }
button:focus-visible { outline: 3px solid #0759bd; outline-offset: 3px; }
```

Run `npm run dev`, open the displayed local URL, and click the button. Expect `Total: 60`. Run `npm run build`, then `npm run preview`. In the preview's network panel, reload with caching disabled and compare requests before and after the click. The report code should be fetched when needed. Inspect `dist` for emitted HTML, hashed JavaScript, and CSS. Generated filenames may differ by tool version, so verify behavior rather than expecting a particular hash.

## Production configuration

If hosting under a subdirectory, configure the build's base URL and verify nested navigation and asset URLs. Environment variables exposed to frontend code become visible to users; they are configuration, not secret storage. Vite exposes client variables with its documented prefix and replaces them during build. A server secret belongs on a trusted server.

Content-hashed assets can receive long immutable cache lifetimes, while HTML usually needs a revalidation policy so clients discover new hashes. Deploy assets before switching the HTML entry point, and retain old chunks long enough for open tabs. Otherwise a user navigating after deployment may request a chunk that was removed.

Source maps aid diagnosis but should follow the application's exposure policy. A build targeting older syntax does not automatically supply every missing browser API; transforms and polyfills solve different compatibility problems. TypeScript transformation is also distinct from type checking, so add a separate checker when needed.

## Debugging and exercises

Inspect errors in order: package resolution, transformation, runtime execution, then network delivery. A successful build with a blank page often points to asset paths or runtime configuration. Measure compressed transfer size and parse/execute time before optimizing a dependency.

1. Add a second optional panel. Acceptance: its code is absent from initial requests and failure shows useful feedback.
2. Deploy the build under `/lab/` in a local host. Acceptance: refresh, styles, and dynamic import all work at that base path.
3. Add a bundle-size budget. Acceptance: the check uses production output and documents whether the budget measures raw, gzip, or Brotli bytes.

## References

- [Vite: building for production](https://vite.dev/guide/build)
- [Vite: environment variables](https://vite.dev/guide/env-and-mode)
- [Vite: static asset handling](https://vite.dev/guide/assets)
