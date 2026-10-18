# Frontend quick reference

[Curriculum](README.md) · [Glossary](GLOSSARY.md) · [Troubleshooting](TROUBLESHOOTING.md)

Use these as reminders after the tutorials. They are not complete applications or substitutes for the surrounding explanations.

## HTML and accessibility

| Need | Preferred starting point | Check |
|---|---|---|
| Navigate to a location | `<a href="…">` | Destination and link text are meaningful |
| Perform an action | `<button type="button">` | Keyboard activation and accessible name |
| Submit a form | `<button type="submit">` | Server destination and duplicate-submit behavior |
| Label a field | `<label for="id">` | Unique matching `id`; placeholder is supplementary |
| Group related controls | `<fieldset><legend>` | Group name explains the relationship |
| Identify primary content | `<main>` | Coherent landmarks and heading hierarchy |
| Explain a field error | Text linked with `aria-describedby` | Set `aria-invalid` when invalid |
| Announce a status | Appropriate live/status region | Avoid excessive or duplicate announcements |
| Present tabular relationships | `<table>`, headers, caption | Row/column relationships are understandable |
| Show a modal | Native dialog or tested dialog pattern | Focus entry, containment, dismissal, restoration |

Prefer native semantics before ARIA. ARIA changes accessibility information, not automatic keyboard behavior. Keep focus visible. Avoid positive `tabindex` values. Test zoom and keyboard operation early.

## CSS decisions

```css
/* Predictable sizing */
*, *::before, *::after { box-sizing: border-box; }

/* Responsive content width */
.container { width: min(100% - 2rem, 70rem); margin-inline: auto; }

/* Content-driven card columns */
.cards { display: grid; gap: 1rem;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr)); }

/* A flex child that may shrink below its content's intrinsic width */
.flex-child { min-width: 0; }

/* Adapt a specific motion feature */
@media (prefers-reduced-motion: reduce) {
  .animated-card { animation: none; transition: none; }
}
```

When a declaration loses, inspect origin/importance, layers, specificity, and source order. When an element overflows, inspect intrinsic sizing, unbroken content, fixed widths, padding, and grid/flex minimums. Do not immediately add `!important` or hide overflow.

Use Flexbox for primarily one-dimensional alignment; use Grid for explicit rows and columns. Use spacing and font tokens for consistency, but keep token names connected to their purpose. Logical properties help with writing direction; they do not automatically localize content.

## JavaScript and data

```js
// Non-mutating updates; nested changed objects need their own copy.
const updated = items.map(item => item.id === targetId ? { ...item, done: true } : item);

// Numeric sorting without mutating the original array.
const ordered = [...items].sort((a, b) => a.rank - b.rank);

// Nullish fallback preserves 0, false, and an empty string.
const pageSize = options.pageSize ?? 20;

// Safe text output for an untrusted display string.
output.textContent = userProvidedText;

// Event listeners can share an explicit lifetime.
const lifetime = new AbortController();
button.addEventListener('click', handleClick, { signal: lifetime.signal });
// When the owning feature is destroyed:
lifetime.abort();
```

Array/object equality normally compares identity, not structural content. Object spread is shallow. `JSON.parse(JSON.stringify(value))` is not a general clone: it changes or rejects several value types. Use the tool that matches the data and required semantics.

`async` returns a promise but does not move CPU work to another thread. Use workers for suitable compute tasks, and account for transfer costs. Do not use `forEach(async …)` when the caller must await completion; choose a sequential loop or a deliberately bounded concurrent strategy.

## Requests and state

```js
async function readJSON(url, signal) {
  const response = await fetch(url, { signal });
  if (!response.ok) throw new Error(`Request failed: ${response.status}`);
  return response.json(); // Validate the resulting data before trusting its shape.
}
```

Fetch rejects for network-level failures and cancellation, not for every HTTP error status. A 204 response has no JSON body to parse. CORS is a server/browser access policy; `mode: 'no-cors'` does not make a readable cross-origin API response.

| State kind | Typical owner |
|---|---|
| Open menu, draft field | Local component or feature |
| Derived filtered results | Compute from source data and filters |
| Shareable filters and page | URL |
| Remote records and freshness | Server-state/query layer |
| Authenticated identity | Server/session contract with client representation |
| Durable device-local notes | IndexedDB with visible persistence status |

Include all server-relevant inputs in cache keys. Invalidate intentionally after mutations. An optimistic update needs failure handling and a concurrency policy. A retry of a mutation may duplicate work unless the server supports safe repetition.

## Component and rendering boundaries

State belongs at the lowest common owner that needs to coordinate it. Pass data and events through an explicit API. Use stable item keys based on identity. Keep side effects out of rendering computations. Clean up subscriptions, timers, streams, object URLs, observers, and workers according to ownership.

SSR creates HTML on a server; hydration connects browser behavior to that HTML. Static generation runs before requests. Client rendering runs in the browser. Server components and server functions have framework-specific rules and still require deliberate data exposure and authorization boundaries.

## Security reminders

Treat URLs, uploaded files, API responses, stored browser data, and query parameters as untrusted. Escape or safely render output for its context. Validate and authorize on the server. Avoid dangerous HTML insertion and unreviewed script execution. Keep private keys out of client environment variables and bundles.

Use session/cookie settings appropriate to the deployment, and design CSRF protection explicitly. CSP is defense in depth; it does not repair every unsafe application operation. Do not log tokens, passwords, or unnecessary personal data. A public client cannot keep a secret embedded in downloaded code.

## Test and performance workflow

1. Define the observable behavior and a realistic failure case.
2. Use the smallest test boundary that proves the behavior.
3. Test a critical complete journey in a browser.
4. Add manual keyboard and assistive-technology checks where interaction warrants them.
5. Measure the production build before optimizing.

Use LCP to investigate major content loading, INP for interaction responsiveness, and CLS for unexpected layout movement. Field and lab data answer different questions. Inspect resource timing and main-thread traces before guessing. Reserve dimensions for media, reduce unnecessary JavaScript, and release resources that outlive their owner.

## Release reminders

Run checks from a clean dependency install, build the intended production target, verify environment values, and test the real deployment URL. Confirm deep links, error pages, cache headers, HTTPS, and identity-dependent behavior. Keep hashed assets compatible with old open tabs. Record the release ID and the rollback path.
