# Web and browser fundamentals

[Curriculum](../README.md)

## Prerequisites and outcomes

No programming experience is required. You need a text editor and a browser with developer tools. The next chapter, [development environment](../02-development-environment-cli-and-git/tutorial.md), explains terminals and local servers. After this tutorial you should be able to trace a page request, distinguish browser responsibilities, inspect a failing resource, and explain why an origin matters.

## How a page reaches the screen

A URL contains a scheme, authority, path, and possibly a query and fragment. In `https://example.com:443/catalog?q=book#results`, HTTPS is the scheme, the host is `example.com`, the path is `/catalog`, and the fragment identifies a location inside the returned document. Fragments normally stay in the browser rather than being sent in the HTTP request. A relative URL is resolved against the document's base URL; moving an HTML file can therefore change what `./app.js` means.

For a network navigation, the browser may resolve a hostname through DNS, establish a connection, negotiate TLS for HTTPS, and send an HTTP request. Existing connections, caches, service workers, or a back/forward cache can change this path. HTTP/1.1, HTTP/2, and HTTP/3 differ in connection and transport behavior; application code should not assume every resource needs a new connection. HTTPS authenticates the server and protects transport; it does not make application code automatically trustworthy.

The response includes a status, headers, and possibly a body. A `200` usually carries content, a `301` or `308` redirects, a `304` supports cache revalidation, and a `404` indicates the requested resource was not found. HTTP failures and network failures are different: a server can successfully deliver an error response. `Content-Type` tells the browser how to interpret bytes; a JavaScript URL accidentally returning an HTML fallback can produce a MIME or syntax error.

The browser parses HTML into a DOM, parses styles into style information, computes styles, determines layout, paints, and composites visual layers. These are useful conceptual stages rather than a guarantee of one pass each. A script can modify the DOM, trigger style changes, or force synchronous layout by reading geometry after writing styles. The accessibility tree exposes a related representation to assistive technologies. Source HTML, the live DOM, pixels, and the accessibility tree are distinct views of the same interface.

An origin is the tuple of scheme, host, and port. `http://localhost:8000` and `http://localhost:9000` are different origins. Same-origin policy restricts some cross-origin interactions, and CORS lets a server permit selected cross-origin reads. CORS is not an authentication system. Cookies, storage, permissions, CSP, mixed-content rules, and secure-context requirements are separate mechanisms. See [MDN's origin explanation](https://developer.mozilla.org/en-US/docs/Glossary/Origin).

## Worked tutorial: inspect a small page

1. Create a folder named `browser-lab`. Add these three files using a text editor.
2. Save `index.html`:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Browser lab</title>
  <link rel="stylesheet" href="styles.css">
  <script src="app.js" defer></script>
</head>
<body>
  <main>
    <h1>Observe the browser</h1>
    <button type="button" id="change">Change message</button>
    <p id="message" role="status">Ready.</p>
  </main>
</body>
</html>
```

Save `styles.css`:

```css
body { font: 1.125rem/1.6 system-ui; margin: 2rem; }
button { font: inherit; padding: .5rem 1rem; }
```

Save `app.js`:

```javascript
const message = document.querySelector('#message');
document.querySelector('#change').addEventListener('click', () => {
  message.textContent = `Updated at ${new Date().toLocaleTimeString()}`;
});
```

3. Opening `index.html` directly demonstrates rendering and interaction. To inspect HTTP, install Python 3 if needed, open a terminal in `browser-lab`, and run:

```powershell
python -m http.server 8000 --bind 127.0.0.1
```

4. Visit `http://127.0.0.1:8000`. Open developer tools, select Network, and reload. Inspect the document, stylesheet, and script requests. An optional favicon request can return 404 without breaking this example.
5. Click the button. Inspect the paragraph in Elements: the live DOM changes, while View Source still shows `Ready.`. Stop the server with Ctrl+C when finished.

The expected result is a styled heading and button whose message updates. `defer` lets the HTML parser finish before this external script executes. `textContent` changes text without interpreting it as markup. No framework or build step is involved.

## Debugging and deeper practice

If the page is blank, inspect Console first, then Network, then the DOM. Check filename capitalization: a development machine may tolerate a mismatch that a production server rejects. Disable cache temporarily while diagnosing a stale asset, but also test with normal cache behavior. A hard reload is not a substitute for a correct caching strategy.

Use throttling to distinguish slow transfer from slow JavaScript. A Performance recording reveals long tasks and layout work. Avoid assuming a delayed screen update means a slow server: the network may already be finished while the main thread remains occupied. Browser processes and threads are implementation details; the practical rule is to keep interaction work short and measure on representative devices.

## Exercises and acceptance criteria

- Break the stylesheet path. Record its status and explain why HTML still appears. Restore the path and confirm styling returns.
- Add a link to `#message`. Confirm clicking it does not produce a document request.
- Serve the same folder on port 8001 in a second terminal. Write both origins and explain why the ports matter.
- Record a reload with cache enabled and disabled. Identify which observations come from evidence rather than assumptions about caching.

## References

- [MDN: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work)
- [MDN: HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [MDN: same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)
- [WHATWG: HTML standard](https://html.spec.whatwg.org/)
