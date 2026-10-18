# Rendering: CSR, SSR, SSG, hydration, and islands

[Curriculum](../README.md)

Prerequisites: [HTML documents](../03-semantic-html/tutorial.md), JavaScript modules, [HTTP](../34-http-fetch-and-rest/tutorial.md), and browser DevTools. Follow this chapter with [Next.js](../42-nextjs/tutorial.md), [Nuxt](../43-nuxt/tutorial.md), [SvelteKit](../44-sveltekit/tutorial.md), or [Astro](../45-astro/tutorial.md).

## Learning outcomes

Choose where and when HTML is generated, distinguish server rendering from interactivity, recognize hydration costs, and compare strategies using network evidence. Rendering is a per-route and often per-component decision; an entire application does not need one universal strategy.

## The rendering choices

Client-side rendering (CSR) builds much of the visible interface after JavaScript loads. Server-side rendering (SSR) generates HTML for a request. Static site generation (SSG) generates HTML ahead of requests, usually during a build. Hydration attaches a framework's client behavior to pre-existing server HTML. SSR can improve initial content availability while still requiring substantial client JavaScript and CPU. Streaming lets a server progressively deliver independent parts of a response. [Rendering on the web](https://web.dev/articles/rendering-on-the-web)

Islands isolate interactive areas within otherwise static HTML. A product article might ship JavaScript only for its calculator and image viewer. Partial hydration changes which regions receive client code; delayed hydration changes when that code starts. Server islands can independently render personalized fragments without making every part of a page equally dynamic. [Astro islands architecture](https://docs.astro.build/en/concepts/islands/)

These strategies have different freshness and hosting costs. A static price becomes stale until a rebuild or revalidation. A request-rendered price can be fresh but depends on backend latency and cache correctness. Personal responses must not accidentally enter a shared public cache. Framework server components are another code-execution boundary; they should not be treated as synonyms for either SSR or islands.

## Worked tutorial: compare response HTML

1. Create `rendering-lab/server.mjs` below. Use a current supported Node.js release and run `node server.mjs`. Open `http://localhost:8080/ssr`, `/ssg`, and `/csr` in separate tabs.

```js
import { createServer } from 'node:http';

const buildTime = new Date().toISOString();
function page(title, body) {
  return `<!doctype html><html lang="en"><meta charset="utf-8">
    <title>${title}</title><main><h1>${title}</h1>${body}</main></html>`;
}
// Startup stands in for a build step: this string is produced once.
const staticPage = page('Static generation', `<p>Built: ${buildTime}</p>`);

createServer((req, res) => {
  res.setHeader('Content-Type', 'text/html; charset=utf-8');
  res.setHeader('Cache-Control', 'no-store');
  const path = new URL(req.url, 'http://localhost:8080').pathname;
  if (path === '/ssg') return res.end(staticPage);
  if (path === '/ssr') {
    return res.end(page('Server rendering', `<p>Requested: ${new Date().toISOString()}</p>`));
  }
  if (path === '/csr') {
    return res.end(page('Client rendering', `
      <p id="result">Loading timestamp…</p>
      <script>
        setTimeout(() => {
          document.querySelector('#result').textContent =
            'Rendered in browser: ' + new Date().toISOString();
        }, 600);
      </script>`));
  }
  res.statusCode = 404;
  res.end(page('Not found', '<p>Try /ssr, /ssg, or /csr.</p>'));
}).listen(8080, '127.0.0.1', () => console.log('http://localhost:8080'));
```

2. Reload each route. The SSG timestamp stays the same until the process restarts; the SSR timestamp changes per request; CSR waits before inserting its timestamp. The artificial delay makes the dependency visible and is not a benchmark.
3. Inspect each document response in Network, then disable JavaScript and reload. The CSR route retains its loading text. SSR and SSG still contain useful content. View Source shows received markup, whereas Elements shows the current DOM after scripts run.
4. Record a small decision table for a documentation article, a private dashboard, and an online editor. Include freshness, personalization, interaction, and hosting requirements. Choose based on those requirements rather than framework fashion.

## Hydration correctness and advanced tradeoffs

Hydration expects the initial client render to agree with the server output. Reading a random value, current time, browser locale, or viewport-dependent branch during the initial render can violate that agreement. Supply stable server data and move genuinely browser-dependent work to the appropriate client lifecycle. Invalid HTML nesting may also be repaired by the browser before hydration begins.

Avoid request waterfalls: fetching a profile before starting an independent catalog request adds unnecessary latency. Parallelize independent data, but keep dependencies sequential. Define loading, error, empty, unauthorized, and not-found behavior per boundary. Streaming improves perceived progress only when the early content is useful and the fallback preserves layout.

Cache keys need to include every input that changes the response. A locale, tenant, role, or experimental variant can change rendered output. Invalidation should have a clear owner and freshness budget. Static generation with incremental regeneration can reduce rebuild costs, but its revalidation behavior is framework- and host-specific.

## Debugging and exercises

Measure the first useful content, JavaScript transfer and execution, interaction delay, and backend time separately. A faster HTML response can coexist with slower interaction after hydration. Test a direct deep link as well as in-app navigation; they often follow different paths.

- Add a static navigation menu to every example. It must work with JavaScript disabled.
- Add a progressively enhanced counter to `/ssg`. The initial markup must remain useful before its event handler is attached.
- Describe a cache-isolation test for a personalized page. Two users must never receive each other's private response.

The example demonstrates timing with plain Node.js, not a production renderer. Primary reading above and [React hydration reference](https://react.dev/reference/react-dom/client/hydrateRoot) explain the next implementation layer. Sources reviewed on 2026-10-18.
