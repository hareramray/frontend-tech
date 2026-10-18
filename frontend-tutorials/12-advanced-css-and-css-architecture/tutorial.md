# Advanced CSS and CSS architecture

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [CSS foundations](../06-css-foundations/tutorial.md), [layout](../07-flexbox-and-grid/tutorial.md), and [responsive design](../08-responsive-design/tutorial.md). You will organize style ownership, use cascade layers and custom properties deliberately, progressively enhance selectors, and compare architectural approaches without confusing tooling with maintainability.

## Make the cascade intentional

Large stylesheets become difficult when ownership is unclear: a component depends on page ancestry, an override depends on import order, and a utility unexpectedly defeats a state. Architecture is the agreement that makes those relationships predictable. Define tokens for shared decisions, components for reusable behavior and appearance, layout rules for composition, and limited utilities for explicit adjustments.

BEM-style naming makes a component's block, element, and modifier visible in class names. CSS Modules scope generated class names through a build tool. Utility-first systems express many decisions directly in markup. CSS-in-JS can colocate styles with behavior but may involve runtime or extraction tradeoffs. Shadow DOM provides a stronger style boundary with its own theming and accessibility considerations. None of these eliminates the need for clear variants, token ownership, and a support policy.

Sass compiles a stylesheet language into CSS and adds facilities such as mixins, functions, and modules. Keep reusable calculations distinct from runtime CSS custom properties: a Sass variable is resolved during compilation, while a custom property can respond to the cascade at runtime. PostCSS runs JavaScript transformations over CSS; selected plugins can add prefixes, transform supported syntax, or enforce conventions. Plugin order, browser targets, generated output, and source maps become part of the build contract. These are different tools that can be combined when their responsibilities are clear. See the [Sass documentation](https://sass-lang.com/documentation/) and [PostCSS project](https://postcss.org/).

Tailwind is one utility-first approach: utility classes and variants compose a design directly in markup. Its strengths include constrained tokens and visible local styling; costs include longer class lists and the need to extract repeated UI into meaningful components. Generated CSS depends on the project's configuration and source detection, so dynamically constructed class fragments require deliberate handling. Follow the documentation for the installed major version rather than mixing setup snippets from different versions. [Tailwind's utility-class guide](https://tailwindcss.com/docs/styling-with-utility-classes) explains the model. CSS Modules preserve conventional stylesheets while limiting naming collisions; CSS-in-JS may add runtime insertion, SSR ordering, or build-time extraction concerns depending on the library. Choose based on the project's rendering model and maintenance needs.

Cascade layers establish precedence before specificity within the relevant origin and importance level. For normal author rules, later layers outrank earlier ones and unlayered rules outrank layered rules. Important declarations reverse layer priority. Declare layer order deliberately and avoid casually mixing layered and unlayered styles. These rules are defined by [the MDN layer reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@layer).

Custom properties are runtime values that inherit and participate in the cascade. They can represent global tokens or component-local parameters. A preprocessor variable is substituted during compilation and has different behavior. Registered custom properties through `@property` can constrain syntax and configure inheritance, but registration changes how invalid values and interpolation behave. Introduce it only when the contract is useful.

## Worked tutorial: a themed component with explicit precedence

1. Create `index.html` and `styles.css` and open the page in a browser that supports cascade layers.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>CSS architecture lab</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
<main class="stack">
  <h1>Notification preferences</h1>
  <section class="panel stack" aria-labelledby="panel-title">
    <h2 id="panel-title">Practice updates</h2>
    <label class="choice">
      <input type="checkbox" name="updates">
      <span>Send learning reminders</span>
    </label>
    <p class="muted">This visual demonstration does not save preferences.</p>
  </section>
</main>
</body>
</html>
```

```css
@layer reset, tokens, base, layout, components, utilities;
@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  body { margin: 0; }
}
@layer tokens {
  :root {
    --color-ink: #18243b;
    --color-muted: #44536c;
    --color-accent: #174ea6;
    --space-unit: 1rem;
  }
}
@layer base {
  body { color: var(--color-ink); font: 1.1rem/1.6 system-ui; padding: 2rem 1rem; }
  h1, h2, p { margin: 0; }
  input { accent-color: var(--color-accent); }
  :focus-visible { outline: 3px solid var(--color-accent); outline-offset: 4px; }
}
@layer layout {
  main { max-inline-size: 38rem; margin-inline: auto; }
  .stack > * + * { margin-block-start: var(--stack-space, var(--space-unit)); }
}
@layer components {
  .panel { --stack-space: .75rem; padding: 1.5rem; border: 2px solid #72849f; border-radius: .75rem; }
  .choice { display: flex; align-items: baseline; gap: .75rem; }
  .panel p { color: var(--color-ink); }
  @supports selector(:has(*)) {
    .panel:has(input:checked) { border-color: var(--color-accent); background: #edf3ff; }
  }
}
@layer utilities {
  .muted { color: var(--color-muted); }
}
```

2. Inspect the paragraph's color. `.muted` wins despite the more specific `.panel p` because its normal declaration is in the later layer.
3. Toggle the checkbox. Supporting browsers change the surrounding panel through `:has()`; other browsers retain a usable checkbox and stable panel.
4. Change `--stack-space` only on the panel. Its internal spacing changes without altering the main stack's spacing.

The expected result is a small component whose state, spacing, and overrides have clear ownership. The checkbox itself remains the source of truth; the CSS state adds a visual cue rather than inventing a parallel state model.

## Advanced features and performance boundaries

`:where()` is useful for low-specificity defaults. `:has()` selects based on relationships, but broad, deeply nested selectors can create unnecessary matching work and unclear dependencies. CSS nesting can improve locality, yet deeply nested rules often reproduce the same specificity problems as long handwritten selectors. Feature queries confirm syntax support; they do not verify every browser implementation detail.

Containment limits how an element's layout, style, or paint effects interact with the rest of the page. `content-visibility: auto` can skip rendering work for offscreen content; pair it with sensible intrinsic-size estimates and test focus, scrolling, find-in-page, and accessibility behavior. Containment is a performance and layout tool with consequences, not a blanket property to apply everywhere.

Organize files around stable responsibilities. Keep global resets small, document token meanings, expose intentional component variants, and remove obsolete overrides when a component changes. A style guide should include content extremes and interaction states, not just perfect default screenshots. Measure unused CSS and delivery size in context; aggressively splitting tiny style files can create overhead without improving user experience.

## Exercises and acceptance criteria

- Add a warning panel variant using a component-local custom property. Global token values must remain unchanged.
- Introduce an unlayered conflicting rule and explain why it wins. Move it into the intended layer and confirm the result.
- Disable the `:has()` block. The control must remain fully usable and understandable.
- Write a style ownership guide covering tokens, components, layouts, utilities, and exceptions. Include one example of each.

## References

- [MDN: cascade layers](https://developer.mozilla.org/en-US/docs/Web/CSS/@layer)
- [MDN: has selector](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)
- [MDN: CSS containment](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_containment)
- [MDN: property registration](https://developer.mozilla.org/en-US/docs/Web/CSS/@property)
- [Sass: documentation](https://sass-lang.com/documentation/)
- [PostCSS: plugin-based processing](https://postcss.org/)
- [Tailwind: utility classes](https://tailwindcss.com/docs/styling-with-utility-classes)
