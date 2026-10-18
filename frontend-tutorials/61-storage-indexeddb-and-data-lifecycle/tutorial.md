# 61. Storage, IndexedDB, and the data lifecycle

[Curriculum](../README.md)

**Prerequisites:** [asynchronous JavaScript](../18-promises-async-and-the-event-loop/tutorial.md) and [security](../40-frontend-security-and-privacy/tutorial.md).

**You will learn to:** choose browser storage, commit a database write correctly, plan migrations, and handle eviction and synchronization.

## Choose storage by behavior

Use memory for temporary state, URL parameters for shareable navigation, `sessionStorage` for small tab-scoped values, and `localStorage` for small persistent string preferences. Web Storage is synchronous: serializing large objects can block interaction. IndexedDB stores structured records asynchronously and supports indexes and transactions. Cache Storage holds request/response pairs; it is useful for resource caching rather than general relational queries. Cookies participate in HTTP requests and have different security and size constraints.

Browser persistence is not a guaranteed backup. Users can clear it, private sessions can restrict it, quotas differ, and the browser may evict data. Explain whether information is saved only on this device or synchronized to an account. Do not describe a successful local write as a successful server sync.

## Tutorial: save and read a note

Create an `index.html` with the usual HTML document structure and `<script type="module" src="notes.js"></script>`. Serve the folder at localhost using the [setup guide](../SETUP.md). Put the following in `notes.js`; output appears in DevTools.

```js
function openNotes() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open("course-notes", 1);
    request.onupgradeneeded = () => {
      request.result.createObjectStore("notes", { keyPath: "id" });
    };
    request.onerror = () => reject(request.error);
    request.onblocked = () => console.warn("Close older tabs to upgrade");
    request.onsuccess = () => {
      const db = request.result;
      db.onversionchange = () => db.close();
      resolve(db);
    };
  });
}

function saveNote(db, note) {
  return new Promise((resolve, reject) => {
    const tx = db.transaction("notes", "readwrite");
    tx.oncomplete = () => resolve();
    tx.onabort = () => reject(tx.error ?? new Error("Write aborted"));
    tx.onerror = () => {}; // The default error behavior aborts the transaction.
    tx.objectStore("notes").put(note);
  });
}

function readNote(db, id) {
  return new Promise((resolve, reject) => {
    const request = db.transaction("notes").objectStore("notes").get(id);
    request.onsuccess = () => resolve(request.result);
    request.onerror = () => reject(request.error);
  });
}

let db;
try {
  db = await openNotes();
  await saveNote(db, { id: "first", text: "Learn transactions", updatedAt: Date.now() });
  console.log(await readNote(db, "first"));
} catch (error) {
  console.error("The note could not be saved", error);
} finally {
  db?.close();
}
```

Expected result: a record with the ID `first` appears in the console and the browser's IndexedDB inspector. Reloading overwrites that same record; it does not append duplicates. Remove the save call and reload to confirm the previously committed data can still be read.

The write resolves on transaction completion rather than on one request's success. A later failure in the transaction can still roll back earlier requests. Start unrelated network requests outside a transaction: IndexedDB transactions can become inactive while control returns to the event loop. Schema changes belong in a version upgrade. See the [IndexedDB lifecycle guide](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB).

## Migrations and synchronization

To add an index, increase the database version and create it during the upgrade only when needed. For multiple historical versions, inspect `event.oldVersion` and apply each required migration. Plan how old open tabs react. Avoid deleting a store to fix a schema error when it contains the user's only copy of data.

Offline synchronization requires an application protocol: record pending operations with stable IDs, retry safely, and make the server deduplicate repeated submissions. Model states such as local-only, queued, syncing, synced, and failed. Use a server revision or another explicit concurrency mechanism to detect conflicting edits. Last-write-wins can discard work; merging text or using CRDTs requires a deliberate data model.

Handle sign-out, account switching, retention, export, and deletion. Prefixing keys by user is insufficient if one account can still read another account's data through application bugs. Clear or partition sensitive caches intentionally and test the transition. Encryption with a key stored beside ciphertext does not protect against malicious JavaScript running in the same origin.

## Debug and practice

Inspect transactions and storage in DevTools. Test a blocked upgrade with two open tabs; test a denied or unavailable storage environment; simulate a failed synchronization. `navigator.storage.estimate()` gives an estimate, not a reservation. A persistence request may be denied and must not be a prerequisite for core usability.

**Exercise:** build a notes list with add, edit, and delete. Acceptance criteria: successful writes survive reload, rejected writes show an error, deleting is explicit, database upgrades retain old notes, and the UI distinguishes local saving from remote synchronization.

**References:** [Storage quotas and eviction](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria), [Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache).
