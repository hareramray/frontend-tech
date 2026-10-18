# Responsive design and adaptive components

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [Flexbox and Grid](../07-flexbox-and-grid/tutorial.md). You will combine intrinsic sizing, viewport media queries, and container queries; choose breakpoints from content; and verify behavior with zoom, varied input devices, and constrained space.

## Responsiveness is more than screen width

The layout viewport is the coordinate space used for CSS layout. A mobile viewport declaration allows the page to use device-width sizing without a desktop-sized virtual layout. Do not disable zoom. Browser zoom, text enlargement, on-screen keyboards, display density, safe areas, and browser chrome all influence the usable area. A small viewport does not reliably identify a phone, and a wide viewport does not prove a mouse is present.

Start with readable content in normal flow. Let images shrink, let text wrap, and use flexible tracks before adding breakpoints. Add a media query when content no longer works well, rather than choosing widths solely because a popular device has that size. A narrow-first stylesheet often needs fewer overrides because the basic one-column layout is already robust.

Viewport media queries are useful for page-wide changes. Container queries let a component respond to its own containing space, so a card can adapt independently in a sidebar or full-width area. A size-query container needs appropriate containment, such as `container-type: inline-size`. The query styles descendants based on an eligible ancestor; an element does not query itself as its own size container. See the [MDN container-query guide](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries).

Preference queries are equally important. `prefers-reduced-motion`, `prefers-color-scheme`, and `forced-colors` support different user needs. `hover` and `pointer` describe capabilities, with `any-hover` and `any-pointer` accounting for additional input mechanisms. Essential actions must remain available without hover. Dynamic viewport units such as `dvh` can reflect changing browser UI, but height constraints should still tolerate larger content.

## Worked tutorial: one component in two containers

1. Create `index.html` and `styles.css`. Open the HTML in a browser. The basic stacked layout works even if container queries are unsupported.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Responsive components</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
<main>
  <h1>Learning opportunities</h1>
  <div class="layout">
    <section class="slot" aria-label="Featured course">
      <article class="promo">
        <div class="art" aria-hidden="true">Aa</div>
        <div><h2>Typography workshop</h2><p>Practice readable, flexible text layouts.</p></div>
      </article>
    </section>
    <aside class="slot" aria-label="Related course">
      <article class="promo">
        <div class="art" aria-hidden="true">01</div>
        <div><h2>Layout basics</h2><p>Understand the space your content needs.</p></div>
      </article>
    </aside>
  </div>
</main>
</body>
</html>
```

```css
* { box-sizing: border-box; }
body { margin: 0; color: #17243b; background: #f5f7fb; font: 1.1rem/1.6 system-ui; }
main { max-inline-size: 80rem; margin-inline: auto; padding: clamp(1rem, 3vw, 3rem); }
h1 { font-size: clamp(1.75rem, 1.25rem + 2vw, 3rem); line-height: 1.15; }
h2 { margin-block: 0 .5rem; line-height: 1.2; }
.layout { display: grid; gap: 1.5rem; }
.slot { container: promotion / inline-size; min-inline-size: 0; }
.promo { display: grid; gap: 1rem; padding: 1.25rem; background: white; border: 1px solid #7a899f; }
.promo p { margin-block-end: 0; }
.art { background: #e0e9ff; padding: 1rem; font-size: 2rem; text-align: center; }
@media (min-width: 60rem) {
  .layout { grid-template-columns: minmax(0, 2fr) minmax(0, 1fr); }
}
@container promotion (min-width: 28rem) {
  .promo { grid-template-columns: 7rem minmax(0, 1fr); align-items: center; }
}
```

2. At a wide viewport, the main grid creates unequal columns. The wider card can become horizontal while the narrower card remains stacked.
3. Resize slowly and observe the exact width where each change occurs. Temporarily change the parent column ratio without editing the component query.
4. Increase browser zoom and text size. The layout should adapt without hiding either title.

The expected result demonstrates two independent decisions: the page chooses its columns from viewport width, while each card chooses its internal layout from available container width. `clamp()` adds bounded fluid typography; it still needs zoom testing because viewport-based terms can dilute text scaling.

## Testing and tradeoffs

Do not test only at named device presets. Sweep continuously through widths, including intermediate values where wrapping is awkward. Check portrait and landscape, long names, translated strings, empty states, and error messages. Use real devices for touch, keyboard appearance, performance, and browser-specific behavior that emulation cannot reproduce fully.

Responsive images use `srcset` and `sizes` for resource selection; CSS width alone does not prevent downloading an oversized image. Art direction uses `picture` to change the image composition itself. Keep reading order stable when moving visual regions. If a dense table needs horizontal scrolling, constrain the scrolling to a named, understandable region rather than forcing the whole page sideways.

Progressive enhancement means the baseline remains usable and advanced features improve it. `@supports` can guard optional layout features. It checks whether syntax is supported, not whether a design is accessible or bug-free. Record a browser support policy and test the actual combinations your audience uses.

## Exercises and acceptance criteria

- Place the same card inside a 20rem-wide wrapper on a wide screen. It must remain stacked.
- Replace every title with a sentence twice as long. No text may overlap or disappear at 320 CSS pixels.
- Add a print stylesheet that removes decorative backgrounds while preserving content.
- Create a test matrix covering narrow/wide, 200% zoom, keyboard, and a touch device. Record observations rather than just checkmarks.

## References

- [MDN: responsive design](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Responsive_Design)
- [MDN: media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries/Using_media_queries)
- [web.dev: viewport units](https://web.dev/blog/viewport-units)
- [MDN: CSS supports queries](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports)
