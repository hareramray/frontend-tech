# 32. Routing and navigation

[Curriculum](../README.md)

Prerequisites: DOM events, URLs, and [modules](../19-modules-errors-and-debugging/tutorial.md).

## What you will learn

Treat the URL as a shareable description of location, implement a small hash router, and account for browser history, focus, titles, loading, and deployment. Understand when a framework router is preferable.

## Concepts and decisions

Routing maps a location to a screen and often to data requirements. Path segments identify resources, query parameters express shareable filters or pagination, and fragments can identify page sections or drive a hash router. Local state such as an unfinished field edit need not always become part of the URL. Choose based on whether a user should bookmark, share, refresh, or revisit the state.

A traditional navigation requests a new document. A client router updates the view inside the current document while maintaining browser history. `history.pushState()` adds a history entry and `replaceState()` updates one; neither automatically renders the route or fires `popstate` for that call. Traversing history produces `popstate`. See [MDN's History API guide](https://developer.mozilla.org/en-US/docs/Web/API/History_API/Working_with_the_History_API).

Hash routing works with simple static hosting because the fragment is not sent in the HTTP request. Path routing needs the host to serve the application's entry document for recognized client routes. API endpoints and missing static assets still need their own correct responses; rewriting every request to HTML can hide failures.

## Worked tutorial: accessible hash navigation

Use a modern browser. Create `index.html` and open it directly. No dependencies are needed:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Notes — Routing lab</title>
<style>
  body { font: 1rem system-ui; max-width: 45rem; margin: 2rem auto; padding: 1rem; }
  nav { display: flex; gap: 1rem; }
  [aria-current="page"] { font-weight: bold; }
</style>
<nav aria-label="Main">
  <a href="#/notes">Notes</a>
  <a href="#/about">About</a>
</nav>
<main id="view"></main>
<script>
  const routes = {
    '/notes': { title: 'Notes', body: 'Your notes will appear here.' },
    '/about': { title: 'About', body: 'This lab keeps navigation in the URL.' }
  };
  const view = document.querySelector('#view');

  function render(moveFocus = false) {
    const path = location.hash.slice(1) || '/notes';
    const page = Object.hasOwn(routes, path) ? routes[path] : {
      title: 'Page not found', body: 'Choose Notes or About from the navigation.'
    };
    const heading = document.createElement('h1');
    heading.textContent = page.title;
    heading.tabIndex = -1;
    const paragraph = document.createElement('p');
    paragraph.textContent = page.body;
    view.replaceChildren(heading, paragraph);
    document.title = `${page.title} — Routing lab`;
    for (const link of document.querySelectorAll('nav a')) {
      if (link.hash.slice(1) === path) link.setAttribute('aria-current', 'page');
      else link.removeAttribute('aria-current');
    }
    if (moveFocus) heading.focus();
  }

  addEventListener('hashchange', () => render(true));
  render();
</script>
</html>
```

Open About, then use Back and Forward. Expect the screen, document title, and current-link indication to stay synchronized. Refresh while on About and confirm the same route appears. Change the fragment to `#/missing` or `#constructor` and expect an explicit not-found screen. Own-property checking prevents inherited object properties from being mistaken for routes. Navigation moves focus to the new heading so keyboard and screen-reader users receive a meaningful location; the initial load does not unnecessarily steal focus.

The router creates text nodes rather than inserting URL-controlled HTML. Route matching is intentionally exact and small. Real applications need structured matching, parameter decoding, nested layouts, error boundaries, lazy loading, and cancellation. Adopt a maintained router when those requirements arise instead of gradually inventing an incomplete framework.

## Data, authorization, and transition behavior

A route can own a data loader so loading and error handling follow navigation. Abort obsolete requests or ignore their results when the user leaves. Decide whether old content remains visible during a transition, whether a pending indicator appears, and what happens when only query parameters change. Preserve form drafts only when the product explicitly expects that behavior.

Client route guards improve navigation experience but do not enforce authorization. The server must verify access to every protected resource. Redirect destinations also need validation so untrusted parameters cannot send users to unintended external sites. Avoid placing secrets or sensitive personal data in URLs because URLs can enter history, logs, and shared links.

Scroll restoration needs a policy. A new page often starts at the top, Back may restore a previous position, and a fragment target may need focused scrolling after content loads. Overriding every browser interaction can break expected behavior. Retain ordinary link semantics for modified clicks, opening a new tab, downloads, and external destinations.

## Debugging and exercises

Test direct entry, refresh, nested paths, unknown paths, Back, Forward, and deployment under a subdirectory. A route that works only after clicking from the home page often indicates server fallback configuration rather than a component bug.

1. Add a search query parameter. Acceptance: the input initializes from the URL, copied URLs reproduce the filter, and Back restores prior filters.
2. Add delayed route data. Acceptance: navigating away prevents stale results from replacing the current screen, and errors have a retry path.
3. Port the example to a framework router. Acceptance: preserve title, focus, not-found behavior, direct entry, and browser-history tests.

## References

- [MDN: URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams)
- [React Router: routing](https://reactrouter.com/start/declarative/routing)
- [Vue Router: getting started](https://router.vuejs.org/guide/)
