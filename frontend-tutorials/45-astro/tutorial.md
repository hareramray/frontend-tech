# Astro: content pages and selective interactivity

[Curriculum](../README.md)

Prerequisites: [semantic HTML](../03-semantic-html/tutorial.md), [React](../27-react/tutorial.md) for this example, and [rendering strategies](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

## Learning outcomes

Build a content-oriented page with a small interactive island, inspect which JavaScript reaches the browser, and choose hydration timing deliberately. Astro is especially useful when most of a page is readable content and only some regions require a component runtime.

## Core concepts

An `.astro` file combines a frontmatter section with a template. Frontmatter executes during rendering, at build time for prerendered pages or on the server for request-rendered pages. It is not automatically browser JavaScript. Components can therefore generate HTML without shipping their implementation to the browser. Scripts explicitly included for browser behavior are a separate concern. [Astro setup](https://docs.astro.build/en/install-and-setup/)

A framework component becomes interactive when a client directive requests hydration. `client:load` starts promptly, `client:idle` defers to idle time, `client:visible` waits for visibility, and `client:media` uses a media condition. `client:only` skips server rendering of that component and needs an appropriate fallback. Choose based on user needs: delaying a primary form's interactivity may be more frustrating than its small download cost. [Framework components](https://docs.astro.build/en/guides/framework-components/)

## Worked tutorial: an article with a reading tracker

1. Create a minimal project and add React support:

```sh
npm create astro@latest reading-guide
cd reading-guide
npm install
npx astro add react
```

Choose a minimal starter and accept the integration changes. Use a Node.js version satisfying the generated project's requirements. Keep the package lockfile so the lab uses consistent dependencies.

2. Create `src/components/ReadingTracker.tsx`:

```tsx
import { useState } from 'react';

export default function ReadingTracker() {
  const [complete, setComplete] = useState(false);
  return (
    <section aria-label="Reading progress">
      <p>{complete ? 'Lesson complete.' : 'Ready to study?'}</p>
      <button
        type="button"
        aria-pressed={complete}
        onClick={() => setComplete(value => !value)}
      >
        {complete ? 'Mark as unfinished' : 'Mark as complete'}
      </button>
    </section>
  );
}
```

3. Replace `src/pages/index.astro`:

```astro
---
import ReadingTracker from '../components/ReadingTracker';
const title = 'Build readable pages first';
const topics = ['Headings', 'Links', 'Forms'];
---
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <meta name="description" content="A short lesson in useful HTML." />
    <title>{title}</title>
  </head>
  <body>
    <main>
      <article>
        <h1>{title}</h1>
        <p>Write content that remains understandable before scripts run.</p>
        <ul>{topics.map(topic => <li>{topic}</li>)}</ul>
      </article>
      <ReadingTracker client:visible />
    </main>
  </body>
</html>
```

4. Run `npm run dev`. The article should render immediately and the visible tracker should become interactive. Toggle it, then reload: the in-memory completion state resets. This example intentionally does not imply persistence.
5. Run `npm run build`, then `npm run preview`. Inspect the built document and Network panel. Remove `client:visible`, rebuild, and compare: the component still renders HTML but its button no longer has React-driven interactivity. Restore the directive before finishing.

## Content, routing, and deployment

Pages under `src/pages` map to routes. Dynamic pages need a defined source of paths when prerendered; request-rendered routes can resolve parameters at runtime. Content collections can validate structured content and provide a consistent interface for authors. Establish content schemas for title, publication state, slug, image alternative text, and dates before the site grows.

Static output is straightforward to cache. Personalized or frequently changing views may need on-demand rendering with an adapter. Server islands offer another way to isolate dynamic server content from the rest of a page. These are deployment choices with cache and privacy consequences, not only syntax changes. [Islands architecture](https://docs.astro.build/en/concepts/islands/)

## Pitfalls and debugging

Two separate islands do not automatically share local component state. If a cart count and cart panel must coordinate, design a shared store, browser event contract, or common island boundary. Avoid loading several framework runtimes without a clear benefit; the ability to mix them does not make that free.

Values passed as client-component props become browser-visible data. Keep secrets out of them. A browser-only dependency may fail during server rendering; isolate it using the integration's supported mechanisms rather than assuming every hydrated component runs only on the client.

Delay hydration only when the fallback remains understandable. A disabled-looking control without explanation can mislead keyboard and assistive-technology users. Verify behavior on slow connections, with scripts unavailable, and while scrolling quickly through several islands. Ensure generated images have dimensions to reduce layout movement.

## Exercises and acceptance criteria

- Convert the article text into a content collection entry. Invalid required metadata must fail validation with a useful message.
- Add a second page using a shared layout. Both pages must have distinct titles and one main content region.
- Compare `client:load` and `client:visible` on a tracker placed below the fold. Record when the component's JavaScript is requested.
- Persist the completion preference locally and handle unavailable storage. The article must remain readable if persistence fails.

Primary Astro documentation linked above was reviewed on 2026-10-18. Generated output and integration defaults should be checked against the installed version.
