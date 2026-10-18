# CSS foundations: cascade, selectors, and the box model

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [semantic HTML](../03-semantic-html/tutorial.md). You will understand how competing declarations are resolved, predict element dimensions, choose useful units, and debug computed styles. This prepares you for [Flexbox and Grid](../07-flexbox-and-grid/tutorial.md).

## The cascade is an ordered decision process

A CSS rule contains a selector and declarations. Selectors match elements; properties set presentation; values may use keywords, numbers, functions, or custom properties. Invalid declarations are generally ignored individually, which enables fallbacks but can hide typos.

When multiple declarations compete, the cascade considers origin and importance, cascade layers, specificity, scoping proximity where applicable, and source order. Specificity is not the first or only decision. Inline styles have special precedence within author styling. `!important` changes cascade priority and reverses layer precedence; it should not become the default way to resolve unclear ownership. Inheritance is separate: some properties, such as text color, inherit by default, while others, such as margin, do not.

Specificity compares ID selectors, class/attribute/pseudo-class selectors, and type/pseudo-element selectors. Combinators do not add specificity. `:where()` contributes zero specificity, while `:is()`, `:not()`, and `:has()` take specificity from their selector arguments. Keep selectors short and meaningful so changes remain local. Browser developer tools show matched rules, overridden declarations, computed values, and inheritance; use that evidence before increasing specificity.

The content box, padding, border, and margin form the box model. With `content-box`, declared width covers content only; with `border-box`, it includes padding and border. Margins remain outside either size. Vertical margins can collapse in normal block flow, which explains some unexpected spacing. Flex and grid containers introduce different formatting behavior.

Use `rem` for sizes tied to the root font, `em` for sizes relative to the relevant font size, percentages where a containing dimension makes sense, and pixels when a precise CSS-pixel measurement is appropriate. A CSS pixel is not necessarily one physical display pixel. Logical properties such as `margin-inline` and `padding-block` follow writing direction and make multilingual layouts easier.

## Worked tutorial: a readable information card

1. Create `index.html` and `styles.css` together. Open the HTML directly in a browser.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>CSS card</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <main>
    <article class="card card--featured">
      <p class="card__eyebrow">Lesson 06</p>
      <h1>Understand the cascade</h1>
      <p>Inspect the rules before adding another override.</p>
      <a class="card__link" href="https://developer.mozilla.org/en-US/docs/Web/CSS">
        Read the CSS reference
      </a>
    </article>
  </main>
</body>
</html>
```

```css
*, *::before, *::after { box-sizing: border-box; }
:root {
  --ink: #152238;
  --surface: #ffffff;
  --accent: #174ea6;
  --space: 1rem;
}
body {
  margin: 0;
  padding: 2rem 1rem;
  background: #edf2f7;
  color: var(--ink);
  font: 1.125rem/1.6 system-ui, sans-serif;
}
.card {
  max-inline-size: 34rem;
  margin-inline: auto;
  padding: calc(var(--space) * 1.5);
  border: 2px solid #77869b;
  border-radius: .75rem;
  background: var(--surface);
}
.card--featured { border-color: var(--accent); }
.card > :first-child { margin-block-start: 0; }
.card > :last-child { margin-block-end: 0; }
.card__eyebrow { font-weight: 700; }
h1 { font-size: 2rem; line-height: 1.2; }
.card__link { color: var(--accent); text-underline-offset: .2em; }
.card__link:focus-visible { outline: 3px solid currentColor; outline-offset: 4px; }
```

2. Inspect the card's box-model diagram. Its maximum width includes the border and padding because of `border-box`.
3. Temporarily move `.card--featured` before `.card`. Their specificity is equal, so the later border shorthand can reset the color. Restore the original order.
4. Change `--space` to `1.5rem`; the padding changes through `calc()` without editing the component rule.

The expected result is a centered card that fits a narrow screen, retains readable line spacing, and shows a visible keyboard focus outline. The custom properties communicate design intent and can be overridden in smaller scopes.

## Debugging and advanced boundaries

Use Computed styles to distinguish a value that lost the cascade from a value that won but produces an unexpected layout. A percentage height may need a definite containing height. `overflow: hidden` can conceal symptoms while clipping content or focus indicators. `z-index` operates within stacking contexts, which can be created by properties such as transforms and opacity; an enormous number does not escape an ancestor stacking context.

The keywords `initial`, `inherit`, `unset`, `revert`, and `revert-layer` have different reset behavior. Shorthands can reset longhands you did not intend to change. Custom properties inherit by default and are resolved where used; a missing variable needs a fallback such as `var(--optional-color, black)`. Later chapters cover layers, containment, feature queries, nesting, and architecture in more depth.

## Exercises and acceptance criteria

- Add a second card variant using a modifier class. Shared spacing must remain in the base class.
- Set the root font size through browser preferences. Text and the card must remain usable without horizontal page overflow.
- Create a specificity conflict, then resolve it by simplifying selectors or changing ownership. Explain why the winning declaration wins.
- Switch the document to `dir="rtl"`. Logical spacing should still work, and the link must remain keyboard accessible.

## References

- [MDN: cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Cascade)
- [MDN: specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_cascade/Specificity)
- [MDN: box sizing](https://developer.mozilla.org/en-US/docs/Web/CSS/box-sizing)
- [MDN: logical properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values)
