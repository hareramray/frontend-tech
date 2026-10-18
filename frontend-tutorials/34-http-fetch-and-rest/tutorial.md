# 34. HTTP, Fetch, and REST

[Curriculum](../README.md)

Prerequisites: promises, [error handling](../19-modules-errors-and-debugging/tutorial.md), and [state management](../33-client-state-management/tutorial.md).

## What you will learn

Read HTTP requests and responses, distinguish transport failures from HTTP errors, cancel obsolete work, and validate response data. Build a browser client and a tiny same-origin local API.

## Concepts and decisions

HTTP requests combine a method, URL, headers, and sometimes a body. Responses carry status, headers, and a body. REST-oriented APIs commonly model resources with URLs and use method semantics consistently: GET reads, POST creates or performs an operation, PUT replaces, PATCH partially changes, and DELETE removes. Actual API contracts still define details such as pagination, conflict responses, and idempotency.

Fetch resolves when it receives an HTTP response, including many unsuccessful statuses. Check `response.ok` or the status explicitly. Reading JSON is asynchronous and can fail if the body is malformed. An `AbortController` can cancel a request and body consumption. CORS is a browser-enforced permission protocol; `mode: 'no-cors'` does not turn a protected response into readable JSON. See [MDN's Fetch guide](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch).

Separate loading, success, empty, and error states. A successful empty array is different from a failed request. Keep the previous usable result when that improves the task, but label stale content if freshness matters. Never silently convert every error into an empty list.

## Worked tutorial: a local catalog endpoint

Use Node.js 22.12 or newer and a browser. Create `server.mjs` and `index.html` in an empty directory. No packages are required. The server is a local learning fixture, not production hosting.

```js
// server.mjs
import { createServer } from 'node:http';
import { readFile } from 'node:fs/promises';

const page = await readFile(new URL('./index.html', import.meta.url));
createServer((request, response) => {
  const url = new URL(request.url, 'http://localhost:3000');
  if (request.method === 'GET' && url.pathname === '/') {
    response.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    response.end(page);
  } else if (request.method === 'GET' && url.pathname === '/api/products') {
    response.writeHead(200, {
      'Content-Type': 'application/json', 'Cache-Control': 'no-store'
    });
    response.end(JSON.stringify([{ id: 'bag', name: 'Canvas bag' }]));
  } else {
    response.writeHead(404, { 'Content-Type': 'application/json' });
    response.end(JSON.stringify({ message: 'Resource not found' }));
  }
}).listen(3000, '127.0.0.1', () => console.log('Open http://127.0.0.1:3000'));
```

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Catalog client</title>
<h1>Catalog</h1>
<button id="load" type="button">Load products</button>
<p id="status" role="status"></p>
<ul id="products"></ul>
<script type="module">
  let controller;
  let sequence = 0;
  const status = document.querySelector('#status');
  const list = document.querySelector('#products');
  document.querySelector('#load').addEventListener('click', async () => {
    controller?.abort();
    controller = new AbortController();
    const currentController = controller;
    const requestId = ++sequence;
    const timeout = setTimeout(() => currentController.abort(), 5000);
    status.textContent = 'Loading…';
    try {
      const response = await fetch('/api/products', {
        signal: currentController.signal, headers: { Accept: 'application/json' }
      });
      if (!response.ok) throw new Error(`Request failed: HTTP ${response.status}`);
      const products = await response.json();
      if (!Array.isArray(products) || products.some(product =>
        !product || typeof product.id !== 'string' || typeof product.name !== 'string'
      )) throw new Error('Unexpected response shape');
      if (requestId !== sequence) return;
      list.replaceChildren(...products.map(product => {
        const item = document.createElement('li');
        item.textContent = product.name;
        return item;
      }));
      status.textContent = products.length ? 'Products loaded.' : 'No products found.';
    } catch (error) {
      if (requestId !== sequence) return;
      status.textContent = error.name === 'AbortError'
        ? 'Request canceled or timed out. Try again.' : error.message;
    } finally {
      clearTimeout(timeout);
    }
  });
</script>
</html>
```

Run `node server.mjs` and open the printed address. Click Load products and expect Canvas bag. Change the fetch path to `/api/missing`, restart the server because it reads HTML at startup, and verify the visible HTTP 404 message. Restore the path, use network throttling, and click twice; only the current request may update the list. Stop the server with Ctrl+C when finished.

## Real API design concerns

Send structured bodies with the correct content type and serialize them explicitly. Handle 204 responses without assuming there is JSON to parse. Distinguish authentication failure, permission denial, validation failure, conflict, rate limiting, and server failure according to the API's documented statuses. Error messages should be useful without exposing internal details.

Pagination may use offsets or cursors. Preserve query parameters in cache keys and URLs when results must be shareable. Retry transient failures only with a bounded policy and respect server retry guidance. A timed-out mutation may have succeeded on the server, so blindly retrying a write can duplicate work.

Cookie authentication requires appropriate server cookie settings and CSRF protection for state-changing requests. Credentialed cross-origin requests require compatible client and server configuration. Do not place API secrets in frontend code. HTTPS protects traffic in deployment; a local HTTP fixture is only a development convenience.

## Exercises and acceptance criteria

1. Add a search parameter using `URLSearchParams`. Acceptance: spaces and punctuation are encoded correctly and the server filters predictably.
2. Add a deliberately malformed response. Acceptance: the client reports a contract error and does not crash while rendering.
3. Add pagination. Acceptance: each page has distinct request identity, old responses cannot overwrite newer pages, and empty results are distinct from failures.

## References

- [MDN: HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
