# Web accessibility

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [semantic HTML](../03-semantic-html/tutorial.md) and [forms](../04-html-forms-and-validation/tutorial.md). You will design keyboard-operable interactions, expose meaningful names and states, accommodate visual and motion preferences, and test an interface beyond automated scans.

## Accessibility as interface behavior

People use keyboards, screen readers, magnification, touch, switches, voice input, high-contrast modes, and combinations of tools. Accessibility is about whether they can understand and complete a task. A checklist helps locate risks, but a passing scanner cannot establish usability. The WCAG principles are perceivable, operable, understandable, and robust; consult the actual success criteria and target level when making a conformance claim.

Semantic HTML supplies roles, relationships, and keyboard behavior. Accessible names identify controls, descriptions add guidance, and states communicate values such as expanded or checked. The visible label should be reflected in the accessible name so speech users can refer to what they see. Prefer native buttons, inputs, links, and dialogs before building custom equivalents. ARIA modifies accessibility semantics; it does not create event handling, focus management, or visual presentation.

Focus is the keyboard's interaction position. Keep a visible focus indicator, preserve a logical DOM order, and avoid positive `tabindex` values that create a separate navigation sequence. `tabindex="0"` adds an element to normal sequential navigation; `tabindex="-1"` permits programmatic focus without normally adding a Tab stop. Hiding a focused element without moving focus can leave users disoriented. New content should not steal focus merely because it loaded.

Color cannot be the only indicator of meaning. Text needs sufficient contrast, controls need discernible boundaries or other identifying information, and zoom must not make essential content inaccessible. WCAG AA commonly requires 4.5:1 for normal text and 3:1 for qualifying large text; these values do not cover every visual requirement. Test reflow, text spacing, orientation, reduced motion, and forced colors against the relevant criteria. [WCAG 2.2](https://www.w3.org/TR/WCAG22/) is the authoritative specification.

## Worked tutorial: an accessible confirmation dialog

1. Create `index.html` with this complete example and open it in a browser with native `dialog` support.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Accessible confirmation</title>
<style>
  body { font: 1.125rem/1.6 system-ui; margin: 2rem; }
  button { font: inherit; padding: .6rem 1rem; }
  :focus-visible { outline: 3px solid #0757b8; outline-offset: 3px; }
  dialog { max-inline-size: min(28rem, 85vw); }
  dialog::backdrop { background: rgb(0 0 0 / .5); }
</style>
<main>
  <h1>Practice workspace</h1>
  <button type="button" id="open">Reset practice progress</button>
  <p id="status" role="status">Your progress is unchanged.</p>
</main>
<dialog id="confirm" aria-labelledby="dialog-title"
  aria-describedby="dialog-description">
  <h2 id="dialog-title">Reset practice progress?</h2>
  <p id="dialog-description">This demonstration changes only the message on this page.</p>
  <form method="dialog">
    <button value="cancel" autofocus>Keep progress</button>
    <button value="confirm">Reset progress</button>
  </form>
</dialog>
<script>
  const trigger = document.querySelector('#open');
  const dialog = document.querySelector('#confirm');
  const status = document.querySelector('#status');
  trigger.addEventListener('click', () => {
    dialog.returnValue = '';
    dialog.showModal();
  });
  dialog.addEventListener('close', () => {
    if (dialog.returnValue === 'confirm') {
      status.textContent = 'Practice progress reset in this demonstration.';
    }
    trigger.focus();
  });
</script>
</html>
```

2. Use only the keyboard. Tab to the trigger, press Enter, and confirm focus starts on the safer action. Tab through the dialog, then press Escape. Focus should return to the trigger.
3. Reopen and choose Reset. The status should update without forcing focus onto the message. Inspect the dialog's name and description in developer tools.

`showModal()` places the dialog in the top layer and makes other document content inert for modal interaction. A `method="dialog"` form closes it and supplies the activated submit button's value. Resetting `returnValue` prevents an earlier confirmed result from being mistaken for a later Escape dismissal. These behaviors are documented in [MDN's modal-dialog API](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement/showModal).

## Testing and tradeoffs

Native dialogs reduce custom behavior, but test the supported browsers and assistive technology combinations. Initial focus should match the task: a destructive confirmation often benefits from a safe initial choice; a long informational dialog may need focus on an introductory element instead. Restore focus to a sensible surviving element if the trigger is removed.

Live regions announce updates without moving focus. A polite status is appropriate for ordinary confirmation; an assertive alert should be reserved for urgent information. Replacing an entire application region on every keystroke can produce excessive announcements. Keep messages brief and relevant.

Combine automated checks with keyboard walkthroughs, screen-reader checks, zoom and reflow tests, contrast inspection, and testing with people who use assistive technology. Record the exact task, expected behavior, observed failure, and environment. Accessibility also applies to loading indicators, validation errors, drag-and-drop alternatives, timeouts, charts, captions, and authentication flows.

## Exercises and acceptance criteria

- Test the dialog at 200% zoom and a narrow viewport. All controls must remain reachable without clipped text.
- Add a second nonmodal help disclosure with native `details`. It must not trap focus or announce itself as a modal.
- Make a five-step keyboard test script that another person can reproduce, including Escape and confirmation paths.
- Inspect the accessibility tree and record the exact names of both dialog buttons. They must match their visible labels.

## References

- [W3C WAI: accessibility tutorials](https://www.w3.org/WAI/tutorials/)
- [WAI-ARIA APG: modal dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/)
- [MDN: accessibility learning material](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Accessibility)
- [W3C: WCAG quick reference](https://www.w3.org/WAI/WCAG22/quickref/)
