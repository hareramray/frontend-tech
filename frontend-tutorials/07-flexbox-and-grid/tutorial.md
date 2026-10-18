# Layout with Flexbox and Grid

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [CSS foundations](../06-css-foundations/tutorial.md). You will choose an appropriate layout model, build a wrapping card collection, align controls inside cards, and diagnose overflow caused by intrinsic sizes.

## Choosing the layout model

Normal flow is the default layout system and remains useful for article text and simple vertical content. Flexbox arranges items along a main axis and distributes space along that axis. Grid defines rows and columns and can align items across both. The common description of Flexbox as one-dimensional and Grid as two-dimensional is a decision aid, not a rule that one cannot wrap or the other cannot create a single row.

In Flexbox, `flex-direction` determines the main axis; `justify-content` works along it, while `align-items` works along the cross axis. `flex-basis` supplies a starting main size, `flex-grow` distributes positive free space, and `flex-shrink` participates when space is insufficient. Shrinking accounts for base size, so equal shrink factors do not imply equal pixel reductions. `gap` creates spacing between items without adding outer margins. `align-content` distributes multiple flex lines only when there is relevant extra cross-axis space.

Grid uses explicit tracks declared by `grid-template-columns` and `grid-template-rows`; automatic placement can create implicit tracks. `fr` distributes available space after other sizing constraints. `minmax(0, 1fr)` permits a track to shrink below its contents' automatic minimum when that is appropriate. `repeat(auto-fit, minmax(...))` builds responsive columns; `auto-fit` collapses unused tracks, while `auto-fill` can preserve empty tracks. Items spanning tracks and intrinsic content can affect the final calculation.

Both systems preserve source order unless you visually reorder items. CSS order and placement do not normally rewrite reading or keyboard order. Build meaningful DOM order first. A layout that looks correct while focus jumps unpredictably is not complete.

## Worked tutorial: course cards with aligned actions

1. Create `index.html` and `styles.css` in an empty folder. Open `index.html` in a browser.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Layout practice</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
<main>
  <header class="page-header">
    <h1>Practice courses</h1>
    <a href="#courses">Browse courses</a>
  </header>
  <div class="courses" id="courses">
    <article class="course">
      <h2>HTML</h2><p>Build meaningful documents.</p>
      <a href="https://developer.mozilla.org/en-US/docs/Web/HTML">Explore HTML</a>
    </article>
    <article class="course">
      <h2>CSS</h2>
      <p>Learn layout, typography, visual states, and flexible component sizing.</p>
      <a href="https://developer.mozilla.org/en-US/docs/Web/CSS">Explore CSS</a>
    </article>
    <article class="course">
      <h2>JavaScript</h2><p>Make useful interactions.</p>
      <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript">Explore JavaScript</a>
    </article>
  </div>
</main>
</body>
</html>
```

```css
* { box-sizing: border-box; }
body { margin: 0; font: 1.1rem/1.6 system-ui; color: #19243b; }
main { max-inline-size: 70rem; margin-inline: auto; padding: 1.5rem; }
.page-header {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: .75rem 2rem;
}
.courses {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 17rem), 1fr));
  gap: 1.25rem;
}
.course {
  display: flex;
  flex-direction: column;
  min-inline-size: 0;
  padding: 1.25rem;
  border: 1px solid #768397;
  border-radius: .75rem;
  overflow-wrap: anywhere;
}
.course h2 { margin-block-start: 0; }
.course a { margin-block-start: auto; color: #174ea6; }
a:focus-visible { outline: 3px solid currentColor; outline-offset: 4px; }
```

2. Resize the viewport. The grid should use as many readable columns as fit and then reduce to one column without a breakpoint.
3. Add several sentences to the CSS card. Within a shared grid row, the links remain at the bottoms of stretched cards because the flex item's auto margin consumes available space.
4. Enable Grid and Flexbox overlays in developer tools. Compare the outer grid tracks to each card's vertical flex layout.

The `min(100%, 17rem)` lower bound prevents the intended card minimum from exceeding a very narrow container. `min-inline-size: 0` and wrapping rules address common intrinsic-size overflow. These choices allow content to fit; they do not justify making text too small.

## Advanced techniques and debugging

Named grid lines and named areas make larger layouts readable, but area names must form valid rectangles. Use `subgrid` when nested items need to participate in a parent's track alignment; verify support for the specific target browsers. A nested independent grid does not automatically inherit its parent's track sizes. Explicitly define implicit row sizing when predictable rows matter.

Absolute positioning removes an element from normal flow and is appropriate for overlays and anchored decorations, not as the general solution for responsive page structure. Sticky positioning needs a threshold such as `top: 0` and interacts with the nearest relevant scrolling ancestor. Overflow on an ancestor can explain why a sticky element behaves unexpectedly.

When a layout overflows, identify the widest element in developer tools before hiding overflow. Common causes include long unbroken strings, fixed-width media, a flex item's automatic minimum, and tracks that cannot shrink. Test empty content, long translations, text zoom, and right-to-left direction. Align by meaningful visual relationships instead of adding arbitrary offsets until one screenshot matches.

## Exercises and acceptance criteria

- Add ten cards and inspect the last row. Explain the difference after replacing `auto-fit` with `auto-fill`.
- Add a long unbroken identifier. No horizontal page scrollbar should appear at a 320 CSS-pixel viewport.
- Build a sidebar/main layout with Grid. At a narrow width it must become one logical column without CSS reordering.
- Replace the header layout with normal flow and explain the user-visible tradeoff. Tab order must stay meaningful in both versions.

## References

- [MDN: Flexbox concepts](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox)
- [MDN: Grid concepts](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout)
- [MDN: subgrid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Subgrid)
- [MDN: grid and accessibility](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Grid_layout_and_accessibility)
