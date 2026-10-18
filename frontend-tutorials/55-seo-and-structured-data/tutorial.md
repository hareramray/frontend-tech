# SEO and structured data

[Curriculum](../README.md)

Prerequisites: [semantic HTML](../03-semantic-html/tutorial.md), [rendering](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md), and [internationalization](../54-internationalization-and-localization/tutorial.md).

## Learning outcomes

Make public content crawlable and understandable, use canonical and indexing controls correctly, add accurate structured data, and verify metadata in the actual delivered document. Search optimization starts with useful content and reliable access, not a collection of hidden keywords.

## Core concepts

Crawling discovers and fetches URLs; indexing decides whether and how content enters a search index; ranking orders results for a query. A page can be reachable but not indexed, and valid structured data does not guarantee a rich result. Metadata should describe the visible page accurately. [Google structured-data introduction](https://developers.google.com/search/docs/appearance/structured-data/intro-structured-data)

Titles, headings, descriptions, meaningful links, image alternatives, and readable content help both people and machines. A canonical URL indicates the preferred version among duplicates; it is not a redirect or an access-control rule. Robots directives influence crawler behavior, but `robots.txt` is public and does not secure private paths. Authentication must protect private content at the server.

## Worked tutorial: a crawlable lesson page

1. Create `seo-lab/index.html` with the following complete example. Serve it with `npx http-server . -p 8080 -c-1` and visit `http://localhost:8080`.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Semantic HTML lesson | Frontend Study</title>
  <meta name="description" content="Learn how headings, landmarks, and meaningful links make an HTML page easier to use.">
  <link rel="canonical" href="https://example.com/lessons/semantic-html/">
  <meta property="og:title" content="Semantic HTML lesson">
  <meta property="og:description" content="Build a page with useful headings, landmarks, and links.">
  <meta property="og:type" content="article">
  <meta property="og:url" content="https://example.com/lessons/semantic-html/">
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Article",
    "headline": "Semantic HTML lesson",
    "description": "Learn how headings, landmarks, and meaningful links make an HTML page easier to use.",
    "inLanguage": "en",
    "mainEntityOfPage": "https://example.com/lessons/semantic-html/"
  }
  </script>
</head>
<body>
  <header><a href="/">Frontend Study</a></header>
  <main>
    <article>
      <h1>Semantic HTML lesson</h1>
      <p>HTML elements describe the purpose of content, not only its appearance.</p>
      <h2>Describe the page structure</h2>
      <p>Use a main landmark for the page's primary content and headings for its sections.</p>
      <h2>Write useful links</h2>
      <p>A link's wording should explain its destination in context.</p>
    </article>
  </main>
</body>
</html>
```

2. Inspect View Source and the Network document response. The heading, article text, metadata, and JSON-LD should be available without waiting for browser scripts. This makes the result easy to inspect even though search engines can process some JavaScript.
3. The `example.com` URLs are teaching placeholders. Before deploying, replace them consistently with the actual public absolute URL. Do not publish a canonical pointing to an unrelated domain. Structured data must describe the real visible content; add authors, dates, or images only when accurate.
4. Validate JSON syntax and use the relevant search engine's structured-data testing tool. A schema-valid object may still lack fields required for a particular search feature. Follow the feature-specific documentation, not only the generic schema vocabulary.

## Routing, indexing, and duplicate control

Serve actual not-found pages with a suitable HTTP status instead of returning the home page with status 200 for every URL. Redirect retired URLs to relevant replacements where appropriate, and avoid long redirect chains. Ensure navigable links use real `href` values; a clickable `div` with JavaScript routing is a poor discovery and accessibility baseline.

Generate a sitemap from canonical public URLs and keep it consistent with redirects and indexing rules. Use `noindex` when appropriate, while understanding that a crawler generally needs to fetch the page to see that directive. Blocking a URL in robots.txt can prevent discovery of its page-level directive. [Google robots and indexing guidance](https://developers.google.com/search/docs/crawling-indexing/robots/intro)

For localized equivalents, use accurate language annotations and reciprocal alternate-language relationships. A locale URL should serve the declared content reliably. Do not silently redirect every visitor based solely on IP when that prevents access to another language version.

## Advanced pitfalls and debugging

Structured data generated from untrusted content needs safe serialization. Inside an HTML script element, a literal closing script sequence can terminate the element even when it appears inside a JSON string. Use a framework-supported safe JSON-LD mechanism or correctly escape HTML-sensitive characters while preserving valid JSON. See [security](../40-frontend-security-and-privacy/tutorial.md).

Social preview metadata and search snippets are related but separate surfaces; each platform may cache previews differently. Check deployed response headers, canonical tags, robots directives, rendered content, mobile usability, and redirect targets when diagnosing a missing page. Do not infer indexing success from a local validation pass.

## Exercises and acceptance criteria

- Add a second lesson with its own title, description, canonical URL, and structured data. None may accidentally reuse the first lesson's identity.
- Generate a sitemap containing only the two intended public canonical routes.
- Create a genuine missing route. Confirm its status and user-facing recovery link.
- Validate structured data against the chosen search feature and document any remaining warnings without inventing missing factual content.

Primary references: [Search Essentials](https://developers.google.com/search/docs/essentials), [Schema.org Article](https://schema.org/Article), and Google guidance linked above. Search documentation reviewed on 2026-10-18; visibility and rich results are never guaranteed.
