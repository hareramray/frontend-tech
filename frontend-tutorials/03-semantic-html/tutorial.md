# Semantic HTML and document structure

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [browser fundamentals](../01-web-and-browser-fundamentals/tutorial.md). You need only a browser and editor. You will build a meaningful document, choose links and buttons correctly, structure repeated information, and inspect the relationship between markup and accessibility.

## Meaning before appearance

HTML describes content and relationships. CSS controls presentation; JavaScript supplies behavior when native HTML does not already provide it. A heading is a heading because it introduces a section, not because it should be large. A navigation landmark groups major navigation. A button performs an action; an anchor with `href` navigates to a resource or location. Replacing a button with a clickable `div` creates extra work for keyboard activation, focus, naming, and disabled behavior.

Start documents with the HTML doctype, a useful title, UTF-8 encoding, the main language, and a viewport declaration. Set `lang` on a passage when its language differs. Headings should reflect a clear hierarchy; do not choose levels for their default font size. A page generally has one principal `main` area. `header` and `footer` can describe either the document or a containing section. `article` fits content that can stand independently, while `section` groups a thematic region normally introduced by a heading. A plain `div` is appropriate when no additional meaning is intended.

Lists represent related items, ordered steps, or term-description relationships through `ul`, `ol`, and `dl`. Use tables for data relationships, with a caption and appropriately associated headers. Do not use tables as the default page layout system. `time` can pair human-readable text with a machine-readable value. `strong` conveys importance; `em` marks emphasis. None of these choices should be driven exclusively by their default appearance.

HTML parsers recover from many errors, which can hide invalid nesting. A browser may repair markup into a DOM different from the source. Inspect the DOM and use a validator when a layout or accessibility relationship behaves strangely. Duplicate IDs, nested interactive controls, and missing labels are especially damaging because scripts and assistive technologies rely on clear relationships.

## Worked tutorial: a course event page

1. Create `index.html` in a new folder. Paste the complete document below.
2. Open it directly in a browser. No server or dependency is required.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Frontend workshop — Learning Club</title>
</head>
<body>
  <a href="#content">Skip to main content</a>
  <header>
    <p>Learning Club</p>
    <nav aria-label="Main">
      <ul>
        <li><a href="#schedule">Schedule</a></li>
        <li><a href="#questions">Questions</a></li>
      </ul>
    </nav>
  </header>
  <main id="content">
    <article aria-labelledby="event-title">
      <h1 id="event-title">Frontend workshop</h1>
      <p>Practice building a useful web page from first principles.</p>
      <p>Date: <time datetime="2027-01-16">16 January 2027</time></p>
      <section id="schedule" aria-labelledby="schedule-title">
        <h2 id="schedule-title">Schedule</h2>
        <table>
          <caption>Workshop sessions, local venue time</caption>
          <thead>
            <tr><th scope="col">Time</th><th scope="col">Session</th></tr>
          </thead>
          <tbody>
            <tr><th scope="row">10:00</th><td>Semantic HTML</td></tr>
            <tr><th scope="row">11:00</th><td>Accessible forms</td></tr>
          </tbody>
        </table>
      </section>
      <section id="questions" aria-labelledby="questions-title">
        <h2 id="questions-title">Common questions</h2>
        <details>
          <summary>Do I need previous experience?</summary>
          <p>No. Bring a computer with a browser and text editor.</p>
        </details>
      </section>
    </article>
  </main>
  <footer><p>Organized by the Learning Club.</p></footer>
</body>
</html>
```

3. Navigate using Tab and Shift+Tab. Activate links with Enter and the disclosure with Enter or Space. The native `details` element already manages its open state.
4. Inspect the accessibility pane in developer tools. Find the heading, main landmark, navigation name, and table header relationships.

The expected result is a fully usable, unstyled document. The schedule link jumps to its section; the question expands without JavaScript. The labels on sections clarify their names when exposed as regions, and the navigation label becomes useful if another navigation area is added.

## Advanced decisions and pitfalls

Native semantics should do most of the work. ARIA can name or describe controls and express patterns missing from HTML, but it does not implement interaction. Adding `role="button"` does not create keyboard behavior. Avoid redundant or conflicting roles. Use `aria-labelledby` when existing visible text supplies a name; use `aria-describedby` for supplementary instructions rather than replacing the name.

Hidden content needs a deliberate choice. `hidden` normally removes an element from rendering and accessibility exposure; visually hiding a label with CSS can preserve it for assistive technology. `aria-hidden="true"` does not prevent keyboard focus, so placing focusable controls inside such a subtree can create confusing experiences. These details are developed in [accessibility](../05-web-accessibility/tutorial.md).

Metadata supports search and sharing, but it cannot rescue poorly organized content. Descriptive titles, meaningful link text, canonical URLs where appropriate, structured data matching visible content, and server-delivered meaningful HTML all contribute to robust documents. Keep machine-readable claims consistent with what people can read.

## Exercises and acceptance criteria

- Add a materials section containing an ordered preparation list. Heading order must remain understandable without styling.
- Add a second navigation region with a distinct accessible name. The two regions must be distinguishable in the accessibility tree.
- Add one data column to the table and verify each value still has a clear header.
- Validate the file with the W3C checker. Resolve errors and explain any remaining warnings in your notes.

## References

- [WHATWG: sections and headings](https://html.spec.whatwg.org/multipage/sections.html)
- [W3C WAI: page structure](https://www.w3.org/WAI/tutorials/page-structure/)
- [W3C WAI: tables](https://www.w3.org/WAI/tutorials/tables/)
- [W3C HTML checker](https://validator.w3.org/nu/)
