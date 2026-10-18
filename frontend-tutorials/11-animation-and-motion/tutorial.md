# Animation and motion

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [CSS foundations](../06-css-foundations/tutorial.md) and [accessibility](../05-web-accessibility/tutorial.md). Basic DOM interaction is introduced here and developed in [DOM events](../17-dom-events-and-interaction/tutorial.md). You will choose a motion technique, implement an interruptible transition, respect reduced-motion preferences, and inspect rendering cost.

## Motion should explain a change

Useful motion communicates cause and effect: an item moves to a new place, a panel becomes available, or a control acknowledges input. Motion can also create distraction, delay, or physical discomfort. Decide what information the animation conveys and ensure that information still exists when the animation is removed.

CSS transitions interpolate when a property changes between states. CSS keyframes describe a sequence that can repeat or run independently. The Web Animations API offers JavaScript control over playback, cancellation, timing, and completion. `requestAnimationFrame()` coordinates custom frame-by-frame updates with rendering opportunities. Choose the least complicated mechanism that expresses the interaction.

Duration controls how long a change takes; easing describes its rate of change. An ease-out transition can make a response feel immediate while settling gradually. An indefinitely repeating animation usually needs a pause or stop mechanism when applicable, and flashing content has separate accessibility risks. Avoid encoding application timing into an arbitrary animation duration: completion of a network operation and completion of decorative motion are different events.

Transforms and opacity often allow efficient compositing, but they are not automatically free. Large layers, expensive shadows, filters, or excessive rasterization can still cost memory and frame time. Animating width, height, or positional layout properties can cause layout work. Measure the actual page and device. `will-change` is an optimization hint to use sparingly after evidence, not a rule to apply to every element.

## Worked tutorial: an interruptible progress marker

1. Create `index.html` and open it in a browser. All code is included below.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Motion practice</title>
<style>
  * { box-sizing: border-box; }
  body { margin: 2rem; font: 1.125rem/1.6 system-ui; color: #18243b; }
  main { max-inline-size: 32rem; }
  button { font: inherit; padding: .5rem 1rem; }
  button:focus-visible { outline: 3px solid #174ea6; outline-offset: 4px; }
  .track { padding: .5rem; margin-block: 1.5rem; background: #e3eafa; overflow: hidden; }
  .marker {
    inline-size: 3rem;
    block-size: 3rem;
    border-radius: 50%;
    background: #174ea6;
    transform: translateX(0);
    transition: transform 350ms cubic-bezier(.2, .8, .2, 1);
  }
  .marker.is-moved { transform: translateX(5rem); }
  @media (prefers-reduced-motion: reduce) {
    .marker { transition: none; }
  }
</style>
<main>
  <h1>Move the practice marker</h1>
  <button type="button" id="move">Move marker right</button>
  <div class="track" aria-hidden="true"><div class="marker"></div></div>
  <p id="status" role="status">Marker is at the left position.</p>
</main>
<script>
  const button = document.querySelector('#move');
  const marker = document.querySelector('.marker');
  const status = document.querySelector('#status');
  let moved = false;
  button.addEventListener('click', () => {
    moved = !moved;
    marker.classList.toggle('is-moved', moved);
    button.textContent = moved ? 'Move marker left' : 'Move marker right';
    status.textContent = moved
      ? 'Marker is at the right position.'
      : 'Marker is at the left position.';
  });
</script>
</html>
```

2. Activate the button repeatedly before a transition finishes. The visual state should reverse smoothly rather than waiting for a queue of animations.
3. Enable reduced motion in the operating system or developer tools. The marker must still move to the correct position, now immediately.
4. Record a short Performance trace while activating the button. Inspect whether the transition triggers repeated layout or mostly rendering/compositing work.

The expected result is an immediate state update with a brief visual transition. The text states the logical result even for someone who cannot see the marker. The decorative track is removed from accessibility exposure, while the status remains available. Focus stays on the initiating control throughout.

## Advanced timing and lifecycle

For frame-based motion, use the timestamp passed to `requestAnimationFrame()` to compute elapsed time. Incrementing a position by a fixed amount per callback makes speed depend on the display refresh rate. Background tabs can throttle callbacks, so clamp very large time deltas or recompute from a stable time origin when appropriate. Keep simulation state separate from rendering state when deterministic behavior matters.

Store animation handles and cancel them when a component is removed or a newer interaction supersedes an old one. With the Web Animations API, cancellation can reject the `finished` promise; handle that path intentionally. A `transitionend` event is not guaranteed when a transition is canceled, removed, or disabled by preferences. Core behavior must not depend solely on it firing.

Animating between measured layouts can use a FLIP approach: measure the first and last geometry, invert the difference with a transform, then animate to the final state. Batch geometry reads and writes to avoid repeated forced layouts. This is an advanced optimization with complexity around scrolling, nested transforms, reduced motion, and interruption.

Do not turn loading spinners into the only indication that work is happening. Provide a text status and a meaningful failure path. Skeletons should reserve reasonable space without pretending the final content has already loaded. Excessive entrance animations can slow scanning, especially for people who repeatedly use the interface.

## Exercises and acceptance criteria

- Add opacity as a second transition while preserving a visible final state. Reduced motion must still produce the correct result.
- Rebuild the marker using the Web Animations API. Rapid input must cancel or replace earlier motion without unhandled rejections.
- Implement a five-second decorative keyframe loop with a pause control. Keyboard users must be able to pause and resume it.
- Compare a transform animation with a layout-property animation in a trace. Report measured differences instead of assuming a winner.

## References

- [MDN: CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_transitions/Using_CSS_transitions)
- [MDN: Web Animations API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API)
- [MDN: requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
- [W3C: animation from interactions](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html)
