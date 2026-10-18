# DOM, events, and interaction

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [semantic HTML](../03-semantic-html/tutorial.md), [forms](../04-html-forms-and-validation/tutorial.md), and [functions](../14-functions-scope-and-closures/tutorial.md). You will create DOM nodes safely, handle native form interaction, delegate repeated events, and manage focus when a control disappears.

## The DOM is the live document

The DOM exposes nodes representing the current document. `querySelector()` returns the first matching element or null; `querySelectorAll()` returns a static NodeList. Other APIs can return live collections that change as the DOM changes. Check your assumptions before modifying a collection during iteration. Attributes describe markup state, while properties can represent current runtime state; for example, an input's current `value` can differ from its original value attribute.

Use `createElement()`, `textContent`, `append()`, `replaceChildren()`, and related APIs to build content. `innerHTML` interprets markup and can create injection vulnerabilities when given untrusted strings. Sanitization is a specialized requirement, not a reason to hand-roll a few string replacements. For repeated structures, a trusted HTML `template` can be cloned and filled with text safely.

Events travel through capture, target, and usually bubble phases. `event.target` identifies the originating target, while `event.currentTarget` identifies the element whose listener is running. Delegation puts one listener on an ancestor and finds an intended descendant, which also works for newly added items. `preventDefault()` cancels a cancelable default action; `stopPropagation()` changes event travel. They solve different problems and should not be added reflexively.

Native controls provide keyboard semantics. Listen for a form's submit event rather than only a button's click so Enter submission works too. Pointer events unify mouse, pen, and touch interactions; dragging still needs keyboard alternatives and cancellation handling. Passive listeners cannot cancel defaults, which matters for scroll-sensitive events.

## Worked tutorial: add and remove practice tasks

1. Create `index.html` and `app.js`. Open the HTML in a browser; this example needs no server.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Practice task list</title>
  <script src="app.js" defer></script>
  <style>
    body { max-inline-size: 40rem; margin: 2rem auto; padding-inline: 1rem; font: 1.1rem/1.6 system-ui; }
    input, button { font: inherit; }
    li { margin-block: .75rem; }
    li button { margin-inline-start: 1rem; }
    :focus-visible { outline: 3px solid #174ea6; outline-offset: 3px; }
  </style>
</head>
<body>
<main>
  <h1>Practice tasks</h1>
  <form id="add-task">
    <label for="title">Task title</label>
    <input id="title" name="title" required maxlength="100" autocomplete="off">
    <button type="submit">Add task</button>
  </form>
  <ul id="tasks" aria-label="Practice tasks"></ul>
  <p id="status" role="status">No tasks yet.</p>
</main>
</body>
</html>
```

```javascript
const form = document.querySelector('#add-task');
const input = document.querySelector('#title');
const list = document.querySelector('#tasks');
const status = document.querySelector('#status');

function announce(action) {
  status.textContent = `${action} ${list.children.length} task(s) in the list.`;
}

input.addEventListener('input', () => input.setCustomValidity(''));

form.addEventListener('submit', (event) => {
  event.preventDefault();
  const title = input.value.trim();
  if (!title) {
    input.setCustomValidity('Enter a title containing more than spaces.');
    input.reportValidity();
    return;
  }
  const item = document.createElement('li');
  const label = document.createElement('span');
  label.textContent = title;
  const remove = document.createElement('button');
  remove.type = 'button';
  remove.dataset.action = 'remove';
  remove.textContent = 'Remove';
  remove.setAttribute('aria-label', `Remove ${title}`);
  item.append(label, remove);
  list.append(item);
  form.reset();
  input.focus();
  announce('Task added.');
});

list.addEventListener('click', (event) => {
  if (!(event.target instanceof Element)) return;
  const button = event.target.closest('button[data-action="remove"]');
  if (!button || !list.contains(button)) return;
  const item = button.closest('li');
  const nextFocus = item.nextElementSibling?.querySelector('button')
    ?? item.previousElementSibling?.querySelector('button')
    ?? input;
  item.remove();
  nextFocus.focus();
  announce('Task removed.');
});
```

2. Add tasks using both the submit button and Enter. Try a title containing `<strong>test</strong>`; it must appear as literal text rather than become markup.
3. Remove the first, middle, and final items using the keyboard. Focus should move to a remaining remove button or return to the title field when the list is empty.

The expected result is an interactive list with one delegated removal listener. The button's accessible name distinguishes repeated Remove actions while retaining the visible word “Remove.” The status reports changes without making the entire list a live region.

## Lifecycle, performance, and advanced patterns

Listeners hold references to their callbacks and captured values. Remove them when their owning component ends, or register them with an AbortSignal and abort the controller during cleanup. A freshly created arrow function is not the same callback reference as an earlier arrow, so it cannot remove that earlier listener by identity. Timers and observers also need lifecycle cleanup.

Batch DOM writes when updating many elements. A DocumentFragment can make assembly convenient, but it is not a universal performance cure. Avoid repeatedly reading layout after writes inside a loop. Use event delegation where it simplifies ownership, not when it makes unrelated controls depend on fragile selector logic.

Shadow DOM can retarget events across boundaries; `composedPath()` helps inspect their path, and event options determine whether custom events cross those boundaries. Custom widgets need documented keyboard behavior and focus management. For complex UI, keep application state separate from its DOM representation so rerendering, persistence, and testing do not depend on reading text back from elements.

## Exercises and acceptance criteria

- Add a completion checkbox to each task. It must have a visible label and work through the keyboard.
- Add a clear-all action with an explicit confirmation. After clearing, focus must land on a useful surviving control.
- Convert listener registration to use one AbortController. After aborting, no owned listener may respond.
- Move tasks into an array with stable IDs and render from that state. Adding and removing must preserve the same accessible behavior.

## References

- [MDN: DOM introduction](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction)
- [MDN: event bubbling](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Event_bubbling)
- [MDN: addEventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)
- [MDN: textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent)
