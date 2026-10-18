# Typography, color, and visual design

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [CSS foundations](../06-css-foundations/tutorial.md), [responsive design](../08-responsive-design/tutorial.md), and [accessibility](../05-web-accessibility/tutorial.md). You will establish readable type, build a small token system, create hierarchy without excessive decoration, and test visual choices against real content.

## Design decisions that support reading

Typography includes font family, size, weight, line height, line length, spacing, and hierarchy. A beautiful typeface can still produce a poor interface if labels wrap unpredictably or paragraphs are too wide. Begin with a system font stack or a small set of licensed fonts, choose a comfortable body size, and limit article width. Around 45–75 characters per line is a useful starting heuristic for prose, not a universal requirement; language, typeface, and reading task change what works.

Unitless line height scales with each element's font size. Avoid fixed-height text containers because zoom and translation change the space text needs. Heading levels express document hierarchy; CSS sizes express visual hierarchy. Use consistent spacing to group related content and separate unrelated content. Repetition reduces the number of visual rules a reader must learn.

Color has roles: foreground, surface, border, action, success, warning, and error. Role-based custom properties let the palette change without rewriting components. Ensure sufficient contrast for the actual adjacent colors, including hover, disabled, selected, and focus states. Do not encode status only as red versus green. Pair color with words, shapes, or icons with appropriate text alternatives.

Hexadecimal and RGB values are familiar, while HSL can simplify hue-based editing. Lab/LCH and OKLab/OKLCH offer different perceptual characteristics useful for palette work, but perceptually uniform color spaces do not automatically guarantee accessible contrast or identical rendering across display gamuts. Provide appropriate fallbacks and inspect the final colors in target browsers. [CSS Color](https://www.w3.org/TR/css-color-4/) defines the color models and conversion behavior.

## Worked tutorial: a readable article with theme tokens

1. Create `index.html` and `styles.css`. Open the page directly in a browser.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Designing for reading</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
<main class="article">
  <p class="eyebrow">Field notes · 5 minute practice</p>
  <h1>Give every paragraph room to breathe</h1>
  <p class="lead">A consistent type system makes a page easier to scan and maintain.</p>
  <h2>Start with the reading task</h2>
  <p>Choose sizes and spacing using realistic content. A short sample heading can hide problems that appear when an editor writes a longer title.</p>
  <p>Use <a href="https://www.w3.org/WAI/tutorials/">accessibility guidance</a> while reviewing the design.</p>
  <aside class="notice" aria-label="Practice note">
    <strong>Practice note:</strong> Check the same page with text enlarged.
  </aside>
</main>
</body>
</html>
```

```css
:root {
  color-scheme: light dark;
  --page: #f7f8fc;
  --ink: #18243b;
  --muted: #43516a;
  --link: #174ea6;
  --notice: #e6edff;
  --border: #6a7c9a;
  --space: 1rem;
}
* { box-sizing: border-box; }
body { margin: 0; background: var(--page); color: var(--ink); font: 1.125rem/1.7 system-ui, sans-serif; }
.article { max-inline-size: 65ch; margin-inline: auto; padding: 3rem 1.25rem; }
h1, h2 { line-height: 1.2; text-wrap: balance; }
h1 { font-size: clamp(2rem, 1.5rem + 2vw, 3.25rem); }
h2 { margin-block-start: 2.5rem; font-size: 1.6rem; }
p { margin-block: var(--space); }
.eyebrow { color: var(--muted); font-size: 1rem; font-weight: 650; }
.lead { font-size: 1.25rem; }
a { color: var(--link); text-underline-offset: .2em; text-decoration-thickness: .1em; }
a:focus-visible { outline: 3px solid currentColor; outline-offset: 4px; }
.notice { padding: 1rem; background: var(--notice); border-inline-start: .3rem solid var(--border); }
@media (prefers-color-scheme: dark) {
  :root {
    --page: #111827;
    --ink: #f1f5fc;
    --muted: #b8c5db;
    --link: #a9c8ff;
    --notice: #22314d;
    --border: #9cb4db;
  }
}
```

2. Toggle the operating system theme or emulate `prefers-color-scheme` in developer tools. Native controls can also follow `color-scheme` when present.
3. Replace the heading with a much longer title and inspect narrow widths. Unsupported `text-wrap: balance` simply leaves normal wrapping.
4. Inspect text contrast with browser tooling and enlarge text. The expected result is a readable article with a clear hierarchy in both themes.

## Fonts, performance, and advanced considerations

If you add a web font, define `@font-face`, provide a suitable modern format, and select an intentional `font-display` strategy. Font swapping can change line breaks and layout. A carefully chosen fallback and metric overrides can reduce shifts. Preload only fonts needed for initial rendering; preloading every weight wastes bandwidth. Variable fonts can combine axes such as weight or width, but compare actual transferred size and used styles rather than assuming they are always smaller.

Use real bold and italic faces when available. Check licensing, supported scripts, diacritics, numerals, and language shaping. `font-variant-numeric: tabular-nums` can align changing numeric columns, while proportional numerals may suit prose. Hyphenation and line breaking need language-aware testing. `letter-spacing` that works in uppercase Latin labels may harm other scripts.

Visual polish should survive forced colors, printing, reduced motion, and missing fonts. Shadows are decorative and should not be the sole means of identifying a control. Test an error message, an empty state, a long localized price, and a crowded toolbar before declaring a design system complete.

## Exercises and acceptance criteria

- Add an error notice with explicit wording and an icon or border cue. It must remain understandable in grayscale.
- Compare two line lengths using the same paragraph. Record which suits reading and which suits scanning, with reasons.
- Add a licensed local font and measure layout shift with and without it. Document fallback behavior.
- Test light, dark, forced colors, and 200% zoom. Content and focus must remain visible in every tested mode.

## References

- [MDN: font-face](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)
- [MDN: font-display](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display)
- [W3C: understanding contrast](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)
- [MDN: prefers-color-scheme](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme)
