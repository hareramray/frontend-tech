# Web performance and Core Web Vitals

[Curriculum](../README.md)

Prerequisites: [rendering strategies](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md), [HTTP](../34-http-fetch-and-rest/tutorial.md), and browser DevTools. Continue with [profiling](../51-profiling-and-memory-management/tutorial.md) and [bundle optimization](../52-code-splitting-and-bundle-optimization/tutorial.md).

## Learning outcomes

Separate loading, responsiveness, and visual stability, collect meaningful performance measurements, and choose improvements based on evidence. Performance is a user experience across devices and networks, not a single score from a powerful development machine.

## Core metrics and interpretation

Core Web Vitals currently comprise Largest Contentful Paint (LCP), Interaction to Next Paint (INP), and Cumulative Layout Shift (CLS). The “good” thresholds are LCP at most 2.5 seconds, INP at most 200 milliseconds, and CLS at most 0.1, assessed at the 75th percentile of visits, with mobile and desktop segmented. LCP reflects major visible content loading, INP interaction responsiveness, and CLS unexpected layout movement. [Web Vitals definitions](https://web.dev/articles/vitals)

Field data represents real visits, including real devices, content, and interaction. Lab measurements provide controlled conditions for diagnosis. A lab run cannot reproduce an entire field distribution, and a newly deployed page may lack enough field samples. Use both. A metric regression can affect one route, locale, browser, or customer cohort while the site-wide average looks healthy.

## Worked tutorial: collect metrics and inspect a slow interaction

1. Create a vanilla Vite lab with a supported Node.js version:

```sh
npm create vite@latest performance-lab -- --template vanilla
cd performance-lab
npm install
npm install web-vitals
```

2. Replace `index.html` with:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Performance lab</title>
<main>
  <h1>Interaction timing</h1>
  <button id="work">Run simulated work</button>
  <p id="result" role="status">Ready</p>
</main>
<script type="module" src="/src/main.js"></script>
</html>
```

3. Replace `src/main.js`:

```js
import { onCLS, onINP, onLCP } from 'web-vitals';

function report(metric) {
  console.table([{
    name: metric.name,
    value: Math.round(metric.value * 1000) / 1000,
    rating: metric.rating,
    id: metric.id,
  }]);
}
onCLS(report, { reportAllChanges: true });
onINP(report, { reportAllChanges: true });
onLCP(report, { reportAllChanges: true });

document.querySelector('#work').onclick = () => {
  // Deliberately bad teaching fixture: do not use busy-waits in real code.
  const end = performance.now() + 250;
  while (performance.now() < end) { /* simulate synchronous work */ }
  document.querySelector('#result').textContent = 'Finished';
};
```

4. Run `npm run build` and `npm run preview`. Open the printed URL, the Console, and the Performance panel. Record clicking the button. You should see a long task and delayed visual feedback. Metrics may update after interaction or when the page becomes hidden; no interaction means there may be no INP value.
5. Remove the busy-wait. Repeat the same recording under the same conditions. The button should respond without the artificial delay. For genuine CPU work, investigate a better algorithm, smaller tasks that yield between useful batches, or a worker. A promise microtask alone does not guarantee a chance to paint.

The measurement library follows browser lifecycle details that a simplistic observer often misses. A single local metric value is diagnostic evidence, not a field percentile. [web-vitals library](https://github.com/GoogleChrome/web-vitals)

## Improvement strategies

For LCP, inspect the time spent waiting for HTML, discovering the main resource, transferring it, and rendering it. Optimize the actual largest element: it might be text, an image, or a different element on mobile. Avoid lazy-loading an above-the-fold LCP image, provide appropriate responsive sources, and prioritize only truly critical resources.

For INP, investigate input delay, handler execution, and presentation delay. Reduce unnecessary synchronous computation, expensive rerenders, layout thrashing, and third-party work. Chunking can improve responsiveness without reducing total CPU time; make sure intermediate work leaves the application in a valid state.

For CLS, reserve space for images, embeds, ads, and asynchronous content. Font loading can change text geometry. Avoid inserting content above a user's current reading position without an appropriate interaction context. A transform animation often avoids layout movement, but its usability and reduced-motion behavior still matter.

## Budgets, pitfalls, and exercises

Set budgets around important journeys: transferred JavaScript, image bytes, main-thread work, and route-specific user metrics. Compare production builds under repeatable conditions and preserve before/after traces. Do not optimize a dependency solely because its package directory is large; shipped compressed code and execution cost are different measures.

Avoid drawing conclusions from one noisy run. Compare several samples, use representative throttling, and watch for extensions or background processes. A perfect lab score does not prove a usable checkout, correct accessibility, or fast API response under load.

- Add an image with explicit dimensions and compare layout stability against an intentionally dimensionless version.
- Measure one real user journey and identify its longest main-thread task with a trace screenshot or exported trace.
- Propose a field collection schema containing metric, route template, release, and device class without personal URLs or form data.
- Define one performance budget and demonstrate how a regression would be detected before release.

Primary metric and library references above were reviewed on 2026-10-18; reevaluate thresholds and browser support when maintaining this chapter.
