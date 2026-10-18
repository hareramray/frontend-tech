# 26. Components and design systems

[Curriculum](../README.md)

Prerequisites: semantic HTML, CSS, and [code quality](../25-code-quality-and-documentation/tutorial.md).

## What you will learn

Design component contracts, separate visual tokens from behavior, and document interaction states. Build a small accessible button family that can become a foundation for a larger system.

## Concepts and decisions

A component packages a repeatable interface and its behavior behind a supported contract. A design system adds shared principles, tokens, patterns, documentation, maintenance, and governance. A folder of styled controls is useful, but it is not the whole system. Reuse should reduce inconsistent decisions rather than merely reduce repeated markup.

Tokens represent design decisions such as spacing, typography, surface colors, border radii, and motion duration. Primitive tokens describe a palette or scale; semantic tokens describe roles such as primary action or danger text. Components consume semantic roles so changing a theme does not require rewriting each selector. Token names should communicate meaning across screens rather than encode a single screenshot's location.

Behavior belongs in the contract. Define allowed variants, size choices, disabled and loading behavior, keyboard interaction, focus visibility, and accessible naming. Prefer native elements when they already provide semantics. A button performs an action; a link navigates. Styling both similarly does not make them interchangeable.

Composition often scales better than dozens of boolean flags. A card can accept a heading, body, and actions without supporting every possible content arrangement through special props. However, unrestricted composition can undermine accessibility, so document required structure and validate critical constraints.

## Worked tutorial: token-driven action controls

Use a modern browser. Create `index.html` in an empty directory; no packages or build steps are required. Open the file directly:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Action component laboratory</title>
<style>
  :root {
    color-scheme: light;
    --color-brand: #174ea6;
    --color-danger: #a51625;
    --color-surface: #fff;
    --color-text: #17202b;
    --color-focus: #6d28d9;
    --space-2: .5rem;
    --space-3: .75rem;
    --radius-control: .4rem;
  }
  body { background: var(--color-surface); color: var(--color-text);
    font: 1rem system-ui; margin: 2rem; }
  .actions { display: flex; flex-wrap: wrap; gap: var(--space-2); }
  .button { --action-color: var(--color-brand); font: inherit;
    min-height: 2.75rem; padding: var(--space-2) var(--space-3);
    border: 2px solid var(--action-color); border-radius: var(--radius-control);
    background: var(--action-color); color: white; cursor: pointer; }
  .button[data-variant="secondary"] { background: transparent;
    color: var(--action-color); }
  .button[data-variant="danger"] { --action-color: var(--color-danger); }
  .button:focus-visible { outline: 3px solid var(--color-focus);
    outline-offset: 3px; }
  .button:disabled { opacity: .6; cursor: not-allowed; }
  @media (forced-colors: active) {
    .button { border-color: ButtonText; }
  }
</style>
<h1>Action controls</h1>
<div class="actions">
  <button class="button" id="save" type="button">Save changes</button>
  <button class="button" data-variant="secondary" type="button">Preview</button>
  <button class="button" data-variant="danger" type="button">Delete draft</button>
</div>
<p id="status" role="status"></p>
<script>
  const save = document.querySelector('#save');
  const status = document.querySelector('#status');
  save.addEventListener('click', async () => {
    save.disabled = true;
    save.setAttribute('aria-busy', 'true');
    status.textContent = 'Saving…';
    await new Promise(resolve => setTimeout(resolve, 800));
    save.disabled = false;
    save.removeAttribute('aria-busy');
    status.textContent = 'Changes saved in this demonstration.';
  });
</script>
</html>
```

The save action is a simulation and does not persist data. The other controls demonstrate appearance only. Expect three visually related variants, visible keyboard focus, and a temporary disabled state while saving. Tab through the controls and activate Save with both Enter and Space. Zoom to 200% and narrow the viewport; the action row should wrap without clipping labels.

## Documenting and evolving the system

Create a component reference that lists its purpose, anatomy, supported variants, examples, and misuse cases. Include long labels, localization, error states, reduced motion, high contrast, and disabled states in review fixtures. A screenshot captures appearance but cannot prove keyboard behavior or accessible naming. Automated checks and manual interaction checks complement each other.

Decide where customization is allowed. Tokens provide broad theme control; explicit variants provide constrained choices; arbitrary CSS overrides can create fragile dependencies on internal markup. Version public changes intentionally. Renaming a token or changing keyboard behavior can break consumers even when a function signature stays the same.

Track adoption and feedback. A component API that repeatedly requires workarounds may be modeling the wrong abstraction. Before generalizing a pattern, compare multiple real use cases and identify the stable common behavior. Prematurely combining unrelated controls creates conditional complexity that every consumer must understand.

## Pitfalls and exercises

Avoid removing outlines without an equivalent focus indication. Disabled controls may be skipped during keyboard navigation; explain unavailable actions nearby when users need to understand why. An icon-only control needs an accessible name, and a tooltip alone may be insufficient. Verify actual contrast for each text/background pair and state.

1. Add a compact size through an explicit modifier. Acceptance: text remains legible, focus remains visible, and the supported hit-area policy is documented.
2. Add a dark theme by changing semantic tokens. Acceptance: component selectors remain unchanged and all variants are reviewed for contrast.
3. Write a component usage page. Acceptance: include keyboard behavior, loading behavior, an accessible icon example, and at least two inappropriate uses.

## References

- [WAI-ARIA Authoring Practices: button pattern](https://www.w3.org/WAI/ARIA/apg/patterns/button/)
- [MDN: CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)
- [Storybook: writing component stories](https://storybook.js.org/docs/writing-stories)
