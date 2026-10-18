# 31. Web components and custom elements

[Curriculum](../README.md)

Prerequisites: [modules](../19-modules-errors-and-debugging/tutorial.md), JavaScript classes, DOM events, and [component contracts](../26-components-and-design-systems/tutorial.md).

## What you will learn

Build an autonomous custom element with a shadow root, lifecycle cleanup, attributes, properties, and a public event. Understand what native component isolation does and does not provide.

## Concepts and decisions

Custom elements let you register a JavaScript class under a hyphenated HTML tag name. The browser constructs instances and invokes lifecycle callbacks. `connectedCallback` runs when an element enters the document, and `disconnectedCallback` runs when it leaves. Observed attributes trigger `attributeChangedCallback`. Setup must tolerate reconnection rather than assuming an element mounts only once. See [MDN's custom-element guide](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_custom_elements).

Shadow DOM provides a separate internal tree and style scoping. It is an encapsulation mechanism, not a security boundary. An open shadow root remains inspectable. CSS custom properties can cross the boundary, and explicit parts can expose intended styling hooks. Slots let consumers supply light-DOM content while the component owns surrounding structure.

Attributes are strings and work naturally in markup. Properties can carry richer values such as objects. If both represent the same setting, define conversion and reflection rules carefully to avoid loops. Public events communicate changes without requiring the component to import its consumers. Native controls inside a custom element retain valuable keyboard and accessibility behavior.

## Worked tutorial: a reusable step counter

Use a modern browser. Create `index.html` in an empty folder and open it directly. No dependencies are required:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Custom counter</title>
<h1>Counter component</h1>
<step-counter step="2"></step-counter>
<p id="notice" role="status">No changes yet.</p>
<script type="module">
  class StepCounter extends HTMLElement {
    static observedAttributes = ['step'];
    #value = 0;
    #controller;
    #button;

    constructor() {
      super();
      const root = this.attachShadow({ mode: 'open' });
      root.innerHTML = `
        <style>
          :host { display: inline-block; }
          button { font: inherit; padding: .6rem 1rem;
            color: var(--counter-color, #174ea6); }
          button:focus-visible { outline: 3px solid currentColor; }
        </style>
        <button type="button" part="button"></button>
      `;
      this.#button = root.querySelector('button');
    }

    get value() { return this.#value; }
    get step() {
      const raw = Number(this.getAttribute('step') ?? 1);
      return Number.isFinite(raw) && raw > 0 ? raw : 1;
    }

    connectedCallback() {
      this.#controller?.abort();
      this.#controller = new AbortController();
      this.#button.addEventListener('click', () => {
        this.#value += this.step;
        this.#render();
        this.dispatchEvent(new CustomEvent('value-change', {
          detail: { value: this.#value }, bubbles: true, composed: true
        }));
      }, { signal: this.#controller.signal });
      this.#render();
    }

    disconnectedCallback() { this.#controller?.abort(); }
    attributeChangedCallback() { this.#render(); }
    #render() {
      this.#button.textContent = `Count ${this.#value}; add ${this.step}`;
    }
  }

  customElements.define('step-counter', StepCounter);
  document.addEventListener('value-change', event => {
    document.querySelector('#notice').textContent = `Count is ${event.detail.value}`;
  });
</script>
</html>
```

Click twice and expect a value of four. In the developer console, change the `step` attribute to `3`; the next click should produce seven. Set it to an invalid string and confirm the documented fallback of one. Remove and reinsert the same element using DOM tools, then click once. It should increment once rather than accumulating duplicate listeners.

The internal markup string is constant. User-provided strings must not be interpolated into HTML without a safe design; `textContent` is used for the changing label. The custom event is dispatched on the host and bubbles to the document. `composed` also allows it to cross any outer shadow boundary containing this component.

## Integration and design tradeoffs

Custom elements work across framework boundaries, but property assignment and event subscription conventions differ between framework versions. Test the intended integration rather than assuming an arbitrary object attribute becomes a JavaScript object. Decide whether the component is controlled by an external value or owns its internal value; mixing both without precedence rules creates synchronization problems.

Shadow DOM can complicate global theme styles, form behavior, testing selectors, and accessible relationships that cross tree boundaries. A form-associated custom element can integrate with forms through `ElementInternals`, but requires a deliberate validity and value contract. When a plain HTML control already fits, wrapping it should provide a concrete benefit.

## Pitfalls and exercises

Do not repeatedly register the same tag name during hot reload without considering the registry. Avoid reading consumer children or attributes for initialization in the constructor; lifecycle callbacks provide the appropriate stage. Moving a node may invoke disconnect/connect callbacks, so cleanup and setup must remain safe.

1. Add a `reset()` method. Acceptance: it updates the visible label and emits the documented event exactly once.
2. Add a consumer theme using `--counter-color` and `::part(button)`. Acceptance: internal selectors remain private and the public styling hooks are documented.
3. Add two instances. Acceptance: their values remain independent, and an ancestor can distinguish event sources through `event.target`.

## References

- [MDN: using shadow DOM](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM)
- [MDN: templates and slots](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_templates_and_slots)
- [MDN: ElementInternals](https://developer.mozilla.org/en-US/docs/Web/API/ElementInternals)
