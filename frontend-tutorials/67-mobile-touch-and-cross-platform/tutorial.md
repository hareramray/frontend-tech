# 67. Mobile, touch, and cross-platform interfaces

[Curriculum](../README.md)

**Prerequisites:** [responsive design](../08-responsive-design/tutorial.md), [events](../17-dom-events-and-interaction/tutorial.md), and [accessibility](../05-web-accessibility/tutorial.md).

**You will learn to:** support several input methods, build a gesture as an enhancement, and assess web, hybrid, and native delivery.

## Design for capabilities

A small screen does not imply touch-only input, and a large screen does not imply a mouse. Users attach keyboards, use styluses, zoom, rotate devices, and operate assistive technologies. Keep every essential action reachable through an explicit control. Hover can reveal supplementary information but should not be the sole route to required content.

Mobile constraints include intermittent connectivity, smaller memory budgets, thermal throttling, dynamic browser chrome, and onscreen keyboards. Test layout while a form field is focused, not just while the keyboard is closed. Use the viewport meta tag without disabling zoom. Consider `dvh`, `svh`, and `lvh` intentionally for full-height layouts, and safe-area environment variables where controls meet device cutouts.

Pointer Events unify mouse, touch, and pen input. Pointer capture keeps an interaction associated with an element while the pointer moves outside it. Cancellation is a normal lifecycle event, and `touch-action` tells the browser which gestures remain native. See the [Pointer Events guide](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events).

## Tutorial: optional swipe navigation

Create a local HTML page with this content. The previous and next buttons are the primary accessible controls; horizontal swiping is an enhancement.

```html
<h1>Travel tips</h1>
<section id="card" aria-label="Tip" style="touch-action:pan-y pinch-zoom;padding:2rem;border:1px solid">
  <p id="tip" aria-live="polite"></p>
</section>
<button id="previous">Previous tip</button>
<button id="next">Next tip</button>
<script type="module">
const tips = ['Pack light.', 'Keep a water bottle.', 'Save your route offline.'];
const card = document.querySelector('#card');
const tip = document.querySelector('#tip');
const previous = document.querySelector('#previous');
const next = document.querySelector('#next');
let index = 0;
let gesture;
function render() {
  tip.textContent = `${index + 1} of ${tips.length}: ${tips[index]}`;
  previous.disabled = index === 0;
  next.disabled = index === tips.length - 1;
}
function move(amount) {
  index = Math.max(0, Math.min(tips.length - 1, index + amount));
  render();
}
previous.addEventListener('click', () => move(-1));
next.addEventListener('click', () => move(1));
card.addEventListener('pointerdown', event => {
  if (!event.isPrimary || event.button !== 0) return;
  gesture = { id: event.pointerId, x: event.clientX, y: event.clientY };
  card.setPointerCapture(event.pointerId);
});
card.addEventListener('pointerup', event => {
  if (!gesture || gesture.id !== event.pointerId) return;
  const dx = event.clientX - gesture.x;
  const dy = event.clientY - gesture.y;
  gesture = undefined;
  if (Math.abs(dx) > 60 && Math.abs(dx) > Math.abs(dy) * 1.5) move(dx < 0 ? 1 : -1);
});
card.addEventListener('pointercancel', () => { gesture = undefined; });
card.addEventListener('lostpointercapture', () => { gesture = undefined; });
render();
</script>
```

Use the [HTML shell](../SETUP.md) around this body fragment. Expected result: buttons and swipes navigate the same bounded list; vertical page scrolling remains available. The threshold avoids treating every slight movement as navigation. Real gesture design must also handle nested links, text selection, edge gestures, and accidental activation; this card intentionally contains no interactive descendants.

## Delivery choices

A responsive web app reaches users through URLs. A PWA can add installation and offline behavior where supported. A hybrid shell, such as Capacitor, embeds web content and bridges selected device capabilities. React Native and similar approaches render native interfaces through a different component/runtime model; ordinary HTML and CSS are not automatically portable. Desktop shells have their own packaging and security boundaries.

Choose based on required device access, distribution, background execution, team experience, accessibility, and maintenance. Share pure domain logic and API types where useful; forcing all navigation and presentation into one abstraction can make each platform worse. Test a bridge failure and permission denial as deliberately as a successful call.

## Debug and practice

Browser device emulation helps with dimensions and throttling but does not reproduce every hardware behavior. Test real devices for scrolling, virtual keyboards, focus, audio, camera, and memory pressure. Inspect passive-event warnings instead of globally disabling browser scrolling. Avoid `touch-action:none` on large content areas unless the product truly needs to own all gestures.

**Exercise:** extend the tips interface with orientation-safe layout and a visible progress indicator. Acceptance criteria: all actions work without swiping, zoom remains available, vertical scrolling works, pointer cancellation leaves no stuck state, and a focused control is not hidden by a virtual keyboard.

**References:** [Viewport meta element](https://developer.mozilla.org/en-US/docs/Web/HTML/Viewport_meta_tag), [Capacitor documentation](https://capacitorjs.com/docs), [React Native introduction](https://reactnative.dev/docs/getting-started).
