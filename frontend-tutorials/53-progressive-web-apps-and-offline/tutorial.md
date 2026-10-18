# Progressive web apps and offline behavior

[Curriculum](../README.md)

Prerequisites: [HTTP](../34-http-fetch-and-rest/tutorial.md), [server state and caching](../35-server-state-and-caching/tutorial.md), and [frontend security](../40-frontend-security-and-privacy/tutorial.md).

## Learning outcomes

Register a service worker, provide a small offline application shell, understand update lifecycle, and distinguish cached reading from reliable offline mutation. A progressive web app combines web capabilities with a usable baseline; installing an icon is only one part of the experience.

## Core concepts

A service worker runs separately from the page and can intercept requests within its scope. It requires a secure context, with localhost available for development. Installation prepares resources, activation handles lifecycle transitions, and fetch events select response strategies. A worker can be stopped and restarted, so its in-memory variables are not durable storage. [Using service workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)

Cache-first favors speed and offline availability for stable assets. Network-first favors fresh content with a cached fallback. Stale-while-revalidate serves cached data while refreshing it in the background. Select a strategy per resource, not one universal strategy for every request. Authenticated responses, financial actions, and user-specific data need particularly careful handling.

## Worked tutorial: an offline reading shell

1. Create `offline-lab` with the following three files. Serve it at the origin root using `npx http-server . -p 8080 -c-1` and visit `http://localhost:8080`.

`index.html`:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Offline lesson</title>
<main>
  <h1>Read this lesson offline</h1>
  <p>Semantic HTML remains useful without a network connection.</p>
  <p id="status" role="status">Preparing offline support…</p>
</main>
<script src="/app.js" defer></script>
</html>
```

`app.js`:

```js
const status = document.querySelector('#status');
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(() => navigator.serviceWorker.ready)
    .then(() => { status.textContent = 'Offline shell installed. Reload once, then test offline.'; })
    .catch(() => { status.textContent = 'Offline support unavailable; online reading still works.'; });
} else {
  status.textContent = 'This browser does not support the offline feature.';
}
```

`sw.js`:

```js
const PREFIX = 'lesson-shell-';
const CACHE = `${PREFIX}v1`;
const ASSETS = ['/index.html', '/app.js'];

self.addEventListener('install', event => {
  event.waitUntil(caches.open(CACHE).then(cache => cache.addAll(ASSETS)));
});

self.addEventListener('activate', event => {
  event.waitUntil(caches.keys().then(keys => Promise.all(
    keys.filter(key => key.startsWith(PREFIX) && key !== CACHE)
      .map(key => caches.delete(key))
  )));
});

self.addEventListener('fetch', event => {
  const request = event.request;
  const url = new URL(request.url);
  if (request.method !== 'GET' || url.origin !== self.location.origin) return;

  if (request.mode === 'navigate' && ['/', '/index.html'].includes(url.pathname)) {
    event.respondWith(fetch(request).catch(async () => {
      const cache = await caches.open(CACHE);
      return (await cache.match('/index.html')) || Response.error();
    }));
  } else if (url.pathname === '/app.js' && !url.search) {
    event.respondWith(caches.open(CACHE).then(async cache =>
      (await cache.match('/app.js')) || fetch(request)
    ));
  }
});
```

2. Wait for installation and reload once so the page becomes controlled. In DevTools Application, inspect the worker and Cache Storage. Enable offline mode and reload: the lesson should remain readable. Uncached unrelated URLs are intentionally not given an arbitrary shell response.
3. Restore online mode. Edit the lesson and change the cache version to `v2`. Observe the worker update and waiting state. Close controlled tabs and reopen to let the update activate naturally. This lab does not force immediate activation because replacing assets underneath an active page can mix releases.

## Installation and durable data

A web app manifest describes names, icons, start URL, scope, display preferences, and colors. Add one when designing an installable experience; provide suitable actual icon files and test each target platform's current installation behavior. A manifest alone does not prove installability or offline readiness. This worked example focuses on offline behavior and does not claim to be a complete installable package. [Web app manifests](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest)

Use IndexedDB for structured offline records that need transactions and asynchronous access. Design queued writes with operation IDs, server-side idempotency, authorization rechecks, retry limits, and conflict resolution. A queued “saved” message should distinguish local persistence from successful synchronization. Background Sync support varies; provide a retry path while the page is open.

## Pitfalls, debugging, and exercises

Never blindly cache every authenticated response or POST operation. Clear account-specific offline data when the product's logout and privacy policy require it. Storage is subject to quota and eviction; offline availability is not an unconditional durability guarantee. `navigator.onLine` is a hint and cannot prove that your backend is reachable.

Use worker-specific logs and inspect response sources. When finished, unregister this lab's worker in DevTools and remove only its `lesson-shell-` caches before reusing localhost port 8080 for another tutorial. Cache cleanup must target your own namespace; deleting every origin cache can break other applications sharing that origin.

- Add a manifest and real icons. Validate display, start URL, and installation on a named supported browser.
- Add an offline banner based on actual request outcomes. It must not promise that an untested backend is reachable.
- Store a draft in IndexedDB and reload offline. Recover the draft and clearly label unsynchronized changes.
- Design an update notification. Accepting an update must preserve unsaved work and avoid an infinite reload loop.

Primary MDN references above were reviewed on 2026-10-18. Recheck platform support before depending on installation, push, or background synchronization capabilities.
