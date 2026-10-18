# Frontend security and privacy

[Curriculum](../README.md)

Prerequisites: [HTML](../03-semantic-html/tutorial.md), [DOM manipulation](../17-dom-events-and-interaction/tutorial.md), [HTTP origins](../34-http-fetch-and-rest/tutorial.md), and JavaScript modules. Continue with [authentication](../39-authentication-and-authorization/tutorial.md) for session-specific design.

## Learning outcomes

Trace untrusted input to dangerous browser operations, render user content safely, distinguish CORS from authorization, and reduce unnecessary collection of personal data. Security work starts by identifying assets, entry points, trust boundaries, and realistic abuse cases.

## Core concepts

Cross-site scripting occurs when attacker-controlled data becomes executable content. Untrusted data can come from query strings, APIs, stored comments, imported files, third-party scripts, or another browser window. HTML escaping is context-dependent: an HTML text context, a JavaScript string, and a URL have different rules. Prefer safe DOM APIs and framework text interpolation. When rich HTML is required, use a maintained sanitizer with an explicit policy, and do not modify sanitized HTML with unsafe string concatenation afterward. [OWASP XSS prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

CORS controls whether a browser exposes a cross-origin response to JavaScript. It does not authenticate a caller or block every cross-origin request. CSRF exploits automatically attached credentials; protect writes on the server. Browser-delivered JavaScript, configuration, source maps, and public environment variables cannot contain secrets. Obfuscation changes readability, not this boundary.

Privacy design asks whether a field or event is needed before asking where to store it. An analytics URL can accidentally include search text or account identifiers. Avoid logging form values, passwords, authentication headers, and unfiltered error objects. Establish retention, deletion, and access rules with the people responsible for the product's requirements; a generic tutorial cannot determine jurisdiction-specific obligations.

## Worked tutorial: a safe profile preview

1. Create `security-lab/index.html` and `security-lab/app.js` with the following contents. Serve the directory with `npx http-server . -p 8080 -c-1`, then open `http://localhost:8080`.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Safe profile preview</title>
<main>
  <h1>Profile preview</h1>
  <form id="profile">
    <label>Name <input name="name" maxlength="80" required></label>
    <label>Website <input name="site" type="url"></label>
    <button>Preview</button>
  </form>
  <p id="message" role="status"></p>
  <section aria-label="Preview"><h2 id="name"></h2><a id="site" hidden>Website</a></section>
</main>
<script type="module" src="app.js"></script>
</html>
```

```js
// app.js
const form = document.querySelector('#profile');
const name = document.querySelector('#name');
const link = document.querySelector('#site');
const message = document.querySelector('#message');

function publicWebsite(raw) {
  if (!raw.trim()) return null;
  const url = new URL(raw);
  if (url.protocol !== 'https:') throw new Error('Use an HTTPS website.');
  if (url.username || url.password) throw new Error('Do not include credentials.');
  return url.href;
}

form.addEventListener('submit', event => {
  event.preventDefault();
  const data = new FormData(form);
  link.hidden = true;
  link.removeAttribute('href');
  name.textContent = String(data.get('name')).trim();
  try {
    const href = publicWebsite(String(data.get('site')));
    if (href) {
      link.href = href;
      link.hidden = false;
    }
    message.textContent = 'Preview updated locally.';
  } catch (error) {
    message.textContent = error.message;
  }
});
```

2. Enter `<strong>Asha</strong>` as the name. The expected result is literal angle brackets and text, with no generated `strong` element. Inspect the DOM to confirm this.
3. Enter an HTTPS website and confirm the link works. Try an HTTP URL: the application rejects it according to this example's deliberate HTTPS-only product policy. `new URL()` parses syntax; the protocol check supplies the policy. This policy does not certify a destination as trustworthy.
4. Inspect Network while submitting. The preview sends no form data anywhere. This is data minimization in observable behavior, not merely a promise in a notice.

## Defense in depth and deployment

A Content Security Policy can restrict script execution and resource destinations. Deploy it as an HTTP response header, start with reporting when necessary, and test realistic user journeys. A nonce must be unpredictable and regenerated per response. Avoid broad wildcard sources and treating `'unsafe-inline'` as a permanent fix. `frame-ancestors` belongs in a response header and can restrict embedding. CSP complements safe rendering; it does not repair arbitrary authorization defects. [OWASP CSP guidance](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)

Review dependencies and third-party scripts as code running with your page's privileges. Lock dependency resolutions, update vulnerable packages deliberately, and verify release changes. Subresource Integrity can protect suitable static external resources, but it is not a general dependency-security solution. Validate `postMessage` origins and message shapes; specify a precise target origin when transmitting sensitive information.

Trusted Types can require designated policies before values reach certain DOM injection sinks in supporting browsers. A policy should perform a deliberate safe transformation, such as reviewed HTML sanitization; a policy that simply returns its input provides no sanitization. Roll out enforcement with reporting and compatibility testing, while retaining safe rendering for browsers without the same support. Dependency provenance, install scripts, lockfile review, and third-party script ownership belong in the same supply-chain review. [MDN Trusted Types](https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API)

## Pitfalls, exercises, and acceptance criteria

Do not replace `textContent` with `innerHTML` to make the example prettier. Client validation improves feedback but must be repeated at the backend boundary. Server-side URL fetching needs additional SSRF defenses; the website-link helper is not sufficient for that job.

- Add a biography textarea. Passing behavior displays multiline text without interpreting markup.
- Document every outgoing origin in a real page. Passing work explains the purpose, data transmitted, and owner of each integration.
- Test a restrictive CSP in a deployed test environment. Passing behavior retains required functionality without broadening the policy blindly.
- Remove personal values from a sample error report. Passing output preserves an actionable error code and release identifier.

Primary references: the OWASP pages above and [MDN same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). Security source review: 2026-10-18.
