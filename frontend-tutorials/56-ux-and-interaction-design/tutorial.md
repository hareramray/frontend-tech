# UX and interaction design for frontend engineers

[Curriculum](../README.md)

Prerequisites: [semantic HTML](../03-semantic-html/tutorial.md), [accessibility](../05-web-accessibility/tutorial.md), and [DOM interaction](../17-dom-events-and-interaction/tutorial.md).

## Learning outcomes

Turn a user task into explicit interface states, provide useful feedback, recover from errors without losing work, and evaluate behavior with realistic tasks. Interaction design is not limited to visual styling: timing, wording, focus, input constraints, and recovery are part of the implementation.

## Core concepts

Start with the task and context. A user saving a project wants confidence that the intended name was saved, not knowledge of HTTP mechanics. Identify the trigger, required input, successful outcome, and likely interruption. Map idle, editing, invalid, submitting, success, and failure states before adding animation or decorative details.

Feedback should distinguish actions the browser has merely received from actions the backend has completed. Prevent accidental duplicate submissions while preserving a recovery path. Explain validation errors near the relevant input, retain valid data, and avoid showing every possible error before the user has had a chance to act. Clear labels and instructions reduce the amount of corrective UI needed. [WAI form notifications](https://www.w3.org/WAI/tutorials/forms/notifications/)

## Worked tutorial: a recoverable save form

1. Save `interaction-lab/index.html` below and open it in a browser. The service is a local simulation, so the result does not persist between reloads.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Save interaction lab</title>
<style>
  main { max-width: 34rem; margin: 2rem auto; padding: 1rem; }
  input, button { font: inherit; padding: .6rem; }
  :focus-visible { outline: 3px solid #075985; outline-offset: 3px; }
  #error { color: #9f1239; }
</style>
<main>
  <h1>Name your project</h1>
  <form novalidate>
    <label for="name">Project name</label>
    <p id="hint">Use between 2 and 40 characters.</p>
    <input id="name" name="name" aria-describedby="hint error" required maxlength="40">
    <p id="error"></p>
    <label><input id="fail" type="checkbox">Simulate a failed save</label>
    <button id="save">Save project</button>
  </form>
  <p id="status" role="status"></p>
</main>
<script type="module">
  const form = document.querySelector('form');
  const name = document.querySelector('#name');
  const save = document.querySelector('#save');
  const error = document.querySelector('#error');
  const status = document.querySelector('#status');
  let pending = false;

  form.addEventListener('submit', async event => {
    event.preventDefault();
    if (pending) return;
    const value = name.value.trim();
    error.textContent = '';
    name.removeAttribute('aria-invalid');
    if (value.length < 2 || value.length > 40) {
      error.textContent = 'Enter a project name between 2 and 40 characters.';
      name.setAttribute('aria-invalid', 'true');
      status.textContent = '';
      name.focus();
      return;
    }
    pending = true;
    save.disabled = true;
    name.readOnly = true;
    const shouldFail = document.querySelector('#fail').checked;
    status.textContent = 'Saving your project…';
    try {
      await new Promise(resolve => setTimeout(resolve, 700));
      if (shouldFail) throw new Error('Simulated failure');
      status.textContent = `Project “${value}” saved in this demo.`;
    } catch {
      status.textContent = 'Your project could not be saved. Your name is still here; try again.';
    } finally {
      pending = false;
      save.disabled = false;
      name.readOnly = false;
    }
  });
</script>
</html>
```

2. Submit an empty form. Focus should move to the field, its error should explain correction, and the input should expose invalid state.
3. Enter a valid name and enable the failure simulation. Saving should show progress, reject duplicate submissions, retain the name, and allow retry. Disable failure and save again. The resulting confirmation should correspond to the submitted value.

## Designing real journeys

For a production service, capture the value associated with a request and decide whether editing remains available during the save. If users can keep editing, indicate when the visible draft differs from the saved version. For operations that are safe to retry, use server idempotency where required; a disabled button alone cannot prevent duplicates caused by network retries or multiple tabs.

Optimistic updates can make low-risk reversible actions feel immediate, but they need rollback or reconciliation. A destructive action may benefit from undo, confirmation, or a staged deletion depending on consequence and reversibility. Do not add confirmation dialogs to every ordinary action; excessive interruption trains users to dismiss them reflexively.

Design empty states with an explanation and a relevant next action. Loading placeholders should preserve structure without pretending content is ready. Error messages should say what happened and what the user can do, without exposing stack traces or blaming the person.

## Evaluation, advanced concerns, and pitfalls

Observe users completing representative tasks with realistic content. Ask for the outcome they are trying to achieve, then watch where they hesitate, repeat actions, or abandon the flow. Task completion, error recovery, time, and qualitative explanations provide different evidence. A button click rate alone may increase because a confusing interface requires repeated clicks.

Test keyboard, touch, screen-reader use, narrow screens, enlarged text, slow networks, interrupted sessions, and longer translations. Motion should clarify state and respect reduced-motion preferences. Preserve focus and scroll appropriately when content changes. Avoid moving a control under the pointer while a user is about to activate it.

## Exercises and acceptance criteria

- Add a successful-save timestamp that clearly reflects the completed request rather than the latest keystroke.
- Replace the fake service with a real endpoint in a test project. Failed saves must preserve the draft and offer retry.
- Conduct a three-person task walkthrough and document observed problems without presenting preferences as universal facts.
- Design an undo flow for an archive action. Undo must restore the actual underlying state and communicate when its time window ends.

Primary references: [WAI forms](https://www.w3.org/WAI/tutorials/forms/), [WAI notifications](https://www.w3.org/WAI/tutorials/forms/notifications/), and [WCAG understanding documents](https://www.w3.org/WAI/WCAG22/Understanding/). These provide technical accessibility guidance; the interaction decisions in this lab are illustrative product choices.
