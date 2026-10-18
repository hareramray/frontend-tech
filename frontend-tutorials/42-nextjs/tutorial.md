# Next.js: routes, server boundaries, and interactive components

[Curriculum](../README.md)

Prerequisites: [React components and state](../27-react/tutorial.md), [TypeScript basics](../20-typescript/tutorial.md), and [rendering strategies](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

## Learning outcomes

Build an App Router page, keep interactive code behind a small client boundary, understand route conventions, and validate the production build. This chapter uses the documented App Router model; framework defaults change, so keep your generated lockfile and consult documentation matching the installed release.

## Core model

Folders under `app` describe route segments. A `page.tsx` exposes a page; `layout.tsx` wraps a subtree and the root layout supplies the document shell. `loading.tsx`, `error.tsx`, and `not-found.tsx` describe distinct states. Route handlers live in `route.ts` and implement HTTP endpoints. [Next.js installation and conventions](https://nextjs.org/docs/app/getting-started/installation)

Pages and layouts are Server Components by default. A `'use client'` directive establishes a client module boundary for state, event handlers, and browser APIs. Its imported dependencies become part of the client graph, so place the boundary near the interaction. A Client Component can still contribute server-rendered HTML on the initial request; the directive does not mean that its first rendering always happens exclusively in the browser. Props crossing the boundary must be supported by React's serialization model. [Server and Client Components](https://nextjs.org/docs/app/getting-started/server-and-client-components)

## Worked tutorial: a reading list

1. Use a currently supported Node.js release satisfying the installer requirements. Run these commands in your projects directory:

```sh
npx create-next-app@latest reading-list
cd reading-list
```

Choose TypeScript, App Router, and **no `src/` directory**. Other formatting and styling choices are optional. If you select `src/`, put every `app/` file below under `src/app/` instead. Keep the generated root layout and stylesheet.

2. Create `app/save-button.tsx`:

```tsx
'use client';

import { useState } from 'react';

export default function SaveButton({ title }: { title: string }) {
  const [saved, setSaved] = useState(false);
  return (
    <button
      type="button"
      aria-pressed={saved}
      onClick={() => setSaved(value => !value)}
    >
      {saved ? `Saved: ${title}` : `Save: ${title}`}
    </button>
  );
}
```

3. Replace `app/page.tsx`:

```tsx
import type { Metadata } from 'next';
import SaveButton from './save-button';

export const metadata: Metadata = {
  title: 'Reading list',
  description: 'A small collection of frontend study topics.',
};

const articles = [
  { id: 'html', title: 'Semantic HTML' },
  { id: 'css', title: 'Responsive CSS' },
];

export default function Page() {
  return (
    <main>
      <h1>Reading list</h1>
      <ul>
        {articles.map(article => (
          <li key={article.id}>
            <h2>{article.title}</h2>
            <SaveButton title={article.title} />
          </li>
        ))}
      </ul>
    </main>
  );
}
```

4. Run `npm run dev` and open the local URL printed by Next.js. Each button should toggle independently and expose its pressed state. Reloading resets the choices because this example intentionally keeps state in memory.
5. Stop the development server, run `npm run build`, then `npm run start`. Verify direct navigation to `/` and inspect the document response: article headings should already be present. A successful development render alone does not verify deployment behavior.

## Data, mutations, and caching

Replace the local array with a server-side data function when connecting a database or API. Keep secrets in server-only modules; data passed to the browser is visible even if its source originated on the server. Fetch independent data concurrently where appropriate and introduce suspense boundaries around meaningful regions.

Treat every Route Handler and Server Action as an externally reachable trust boundary. Validate inputs and enforce session, resource ownership, and permission checks at the operation itself. A hidden button or an earlier redirect is not sufficient. See [authentication](../39-authentication-and-authorization/tutorial.md).

Next.js caching has several interacting layers and has evolved across releases. Decide which data may be cached, for whom, and for how long. Follow the installed version's documentation for explicit cache controls, cache components, and invalidation. Never assume a default caching rule recalled from another major version. [Next.js caching](https://nextjs.org/docs/app/getting-started/caching)

## Pitfalls and debugging

Hydration mismatches often come from non-deterministic initial output or invalid markup. A browser-global reference can also fail during server rendering even inside a client module; put browser-only work in a suitable effect or event handler. Do not promote the entire root layout to a client boundary merely to fix one interactive control.

For dynamic routes, use the parameter contract of your installed release; modern examples await route parameters. Diagnose errors from server logs and browser logs separately. Run the configured linter explicitly: a production build is not a substitute for every quality check. Deployment must provide the runtime needed by your selected server features; a static export cannot implement arbitrary request-time backend behavior.

## Exercises and acceptance criteria

- Add a detail page and use `next/link` to navigate. Directly opening its URL must also work.
- Add an empty-state message when the array contains no articles. The page must still have one clear main heading.
- Persist saved articles through a real authorized endpoint. Unauthenticated writes must fail server-side, and failed writes must restore an accurate button state.
- Compare browser JavaScript before and after importing a large library into `SaveButton`. Explain why its dependency becomes client work.

Official framework sources linked in this chapter were reviewed on 2026-10-18.
