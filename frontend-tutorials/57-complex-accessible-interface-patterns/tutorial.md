# Complex accessible interface patterns

[Curriculum](../README.md)

Prerequisites: [web accessibility](../05-web-accessibility/tutorial.md), [DOM events](../17-dom-events-and-interaction/tutorial.md), and [accessibility testing](../49-accessibility-testing/tutorial.md).

## Learning outcomes

Choose an appropriate widget pattern, manage focus deliberately, distinguish semantic roles from keyboard implementation, and recognize the accessibility costs of virtualization and drag-and-drop. A visually similar popup may be a disclosure, menu, listbox, combobox, or dialog; these are different interaction contracts.

## Pattern selection

A disclosure reveals content from a button, usually with `aria-expanded` and a relationship to the panel. A menu represents commands with a menu keyboard model; ordinary website navigation is usually a list of links, not an application menu. A listbox selects options, while a combobox combines an input or trigger with a related popup. Tabs switch among associated panels. Grids support two-dimensional interaction and should not be added merely to describe a CSS grid layout.

ARIA roles and properties communicate semantics; they do not implement arrow-key navigation, focus management, selection, or dismissal. Prefer native controls where their behavior matches the task. For a custom composite, follow the relevant WAI-ARIA Authoring Practices pattern and test the actual browser/assistive-technology combinations. [APG patterns](https://www.w3.org/WAI/ARIA/apg/patterns/)

## Worked tutorial: a native rename dialog

1. Save `dialog-lab/index.html` below and open it in a modern browser supporting `<dialog>`. No dependencies are required.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Accessible rename dialog</title>
<style>
  dialog { max-inline-size: 30rem; border: 1px solid; padding: 1.5rem; }
  dialog::backdrop { background: rgb(0 0 0 / .45); }
  :focus-visible { outline: 3px solid #075985; outline-offset: 3px; }
</style>
<main>
  <h1 id="project">Study project</h1>
  <button id="rename">Rename project</button>
  <p id="status" role="status"></p>
</main>
<dialog id="dialog" aria-labelledby="dialog-title">
  <form method="dialog">
    <h2 id="dialog-title">Rename project</h2>
    <label for="new-name">New project name</label>
    <input id="new-name" name="name" required maxlength="60" autofocus>
    <button value="save">Save name</button>
    <button id="cancel" type="button">Cancel</button>
  </form>
</dialog>
<script type="module">
  const dialog = document.querySelector('#dialog');
  const trigger = document.querySelector('#rename');
  const input = document.querySelector('#new-name');
  const project = document.querySelector('#project');
  let opener;

  trigger.onclick = () => {
    opener = document.activeElement;
    input.value = project.textContent;
    input.setCustomValidity('');
    dialog.returnValue = '';
    dialog.showModal();
    input.select();
  };
  input.addEventListener('input', () => input.setCustomValidity(''));
  dialog.querySelector('form').addEventListener('submit', event => {
    if (!input.value.trim()) {
      event.preventDefault();
      input.setCustomValidity('Enter a project name.');
      input.reportValidity();
    }
  });
  document.querySelector('#cancel').onclick = () => dialog.close('cancel');
  dialog.addEventListener('close', () => {
    if (dialog.returnValue === 'save') {
      project.textContent = input.value.trim();
      document.querySelector('#status').textContent = 'Project renamed.';
    }
    if (opener?.isConnected) opener.focus();
  });
</script>
</html>
```

2. Open the dialog using the keyboard. Focus should enter the name field; Tab and Shift+Tab should remain within the modal controls. Escape or Cancel should close it without changing the project. Saving should update the heading and restore focus to the opener.
3. Try an empty or whitespace-only name. The dialog must stay open and identify the invalid field. Inspect the accessibility tree: the dialog should have the name “Rename project.” Native modal behavior handles background inertness; avoid adding redundant custom focus traps that conflict with it. [MDN dialog](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/dialog)

## Focus models and advanced widgets

Roving tabindex keeps one item in a composite in the normal tab sequence while arrow keys move focus among items. `aria-activedescendant` keeps DOM focus on a controlling element while identifying the active descendant. Both require careful synchronization of active, selected, disabled, and expanded states. For a combobox, typing, input-method composition, filtering, popup selection, Escape, and blur can interact; use a well-tested implementation when the requirements are substantial.

Modal dialogs need a sensible initial focus target and an accessible name. A long explanatory dialog may initially focus a heading or introductory element rather than its first destructive action. Return focus to the triggering control when it still exists, otherwise to a logical place in the resulting workflow. [APG modal dialog](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)

Virtualized lists render only part of a dataset. This reduces DOM cost but can remove the focused item, confuse item counts, break browser find, and hide content from assistive navigation. Preserve stable identity and focused items, expose appropriate position/size information for the chosen pattern, and test scrolling with keyboard and screen reader. Pagination may be a better tradeoff for some tasks.

Drag-and-drop needs an alternative usable without precise dragging. Provide Move up/down controls or a destination picker, announce the result, and preserve focus on the moved item. Keyboard support alone is not the only concern: a pointer user with limited dexterity may also need a non-drag action. Avoid relying on deprecated drag ARIA properties.

## Pitfalls and exercises

- Implement tabs using the APG contract. Each tab must control its panel, arrow navigation must be predictable, and only the active tab should be in the composite's tab stop pattern.
- Add a reorderable list with Move up/down buttons. Its announcement must identify the item and final position.
- Test a virtualized list across a window boundary. Focus must not disappear when the active row leaves the ordinary rendered slice.
- Evaluate the dialog with a screen reader and at enlarged text size. All controls must remain reachable and the name must be announced clearly.

Primary W3C and MDN pattern references above were reviewed on 2026-10-18. Automated checks supplement, but cannot replace, interaction testing.
