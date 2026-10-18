# Project 03. Build offline notes with honest save states

[Projects](../README.md) · [Curriculum](../../README.md)

**Prerequisites:** [async JavaScript](../../18-promises-async-and-the-event-loop/tutorial.md), [IndexedDB](../../61-storage-indexeddb-and-data-lifecycle/tutorial.md), [PWA lifecycle](../../53-progressive-web-apps-and-offline/tutorial.md), and [security](../../40-frontend-security-and-privacy/tutorial.md).

**Outcome:** a device-local notes application that survives reloads and works offline after its shell is cached. Synchronization is an optional later extension, not an implied feature.

## Define the promise to users

Display “Saved on this device” after a committed IndexedDB transaction. Do not say “Backed up” or “Synced” unless a remote service has acknowledged the data. Explain that clearing browser data can erase local notes, and provide export before the app becomes useful enough to hold important work.

Use a record such as `{ id, title, body, createdAt, updatedAt, revision }`. Generate stable IDs with `crypto.randomUUID()` in an eligible environment. Store plain text first; rich text introduces a separate document model, sanitization policy, and editing lifecycle.

## Milestone 1: persistent CRUD

Adapt the database-open and transaction functions from the IndexedDB chapter into a `storage.js` module. Add list, read, write, and delete operations. Render names and note content through text nodes, not HTML interpretation.

Design a sidebar list, a labeled title input, a labeled body textarea, and explicit save/delete controls. Start with explicit Save so persistence behavior is easy to inspect. Keep the previous saved record separate from the current draft. A failed write must leave the draft available for copying or retrying.

**Check:** create a note, reload, edit it, reload again, and verify the latest committed value. Delete only after a deliberate user action. Simulate a rejected write and confirm the UI does not claim success.

## Milestone 2: autosave without lying

After explicit saving works, add debounced autosave. Track `dirty`, `saving`, `saved`, and `failed` states, with a draft revision number. When a save resolves, compare the saved revision with the current draft revision: an older successful write does not mean the current draft is saved.

Serialize writes per note or otherwise guarantee their intended order. A debounce timer controls when work starts; it does not solve an in-flight write race. Do not rely on an asynchronous write in `beforeunload` to save the final edit. Save during normal use and warn about unsaved work only when there is a meaningful risk.

**Check:** type while a save is artificially delayed. The interface must remain dirty or saving until the newest revision commits. A reload after the final saved indication must reproduce the latest text.

## Milestone 3: cache the application shell

Use a fixed initial shell: `index.html`, `styles.css`, `app.js`, and `storage.js`. The following `sw.js` is a teaching baseline for a small static app:

```js
const CACHE = 'notes-shell-v1';
const SHELL = ['./', './index.html', './styles.css', './app.js', './storage.js'];
self.addEventListener('install', event => {
  event.waitUntil(caches.open(CACHE).then(cache => cache.addAll(SHELL)));
});
self.addEventListener('activate', event => {
  event.waitUntil(caches.keys().then(keys => Promise.all(
    keys.filter(key => key.startsWith('notes-shell-') && key !== CACHE)
      .map(key => caches.delete(key))
  )));
});
self.addEventListener('fetch', event => {
  const url = new URL(event.request.url);
  if (event.request.method !== 'GET' || url.origin !== self.location.origin) return;
  event.respondWith(caches.open(CACHE).then(async cache => {
    const cached = await cache.match(event.request);
    return cached || fetch(event.request);
  }));
});
```

Register it from `app.js` with `navigator.serviceWorker.register('./sw.js')` behind a support check, and handle registration errors. This example caches only the listed shell, not arbitrary responses or note data. Every listed file must exist or installation fails. A versioned-cache update takes effect through the normal service-worker lifecycle; do not blindly force activation while older clients depend on older assets.

**Check:** open online, allow installation, reload so the page is controlled, then test offline. Confirm the shell and saved notes load. A first-ever offline visit cannot download a shell it has never received.

## Milestone 4: migration, export, and recovery

Add JSON export with a format version and a visible download action. Import must validate structure, limit size, and define whether duplicate IDs replace, merge, or create records. Preview the effect before a destructive replacement. Never treat imported HTML as trusted markup.

Create a database migration that adds an index without losing existing notes. Test two tabs during the upgrade. Add a visible path to retry or reload when another tab blocks the upgrade. Define how simultaneous edits across tabs are detected; a simple app can warn about a conflicting revision instead of silently overwriting it.

**Completion criteria:** notes survive reload; offline use works after installation; save status reflects committed data; failures preserve drafts; upgrades retain notes; export and validated import work; and clearing browser storage is clearly understood as data loss without a remote backup.

**Stretch work:** add a remote sync service with operation IDs, deduplication, authentication, conflict handling, and explicit local-versus-remote status. Do not label a queue “synced” merely because a network request was attempted.
