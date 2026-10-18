# HTML forms and validation

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [semantic HTML](../03-semantic-html/tutorial.md). The JavaScript in this tutorial is explained where used; revisit it after [DOM events](../17-dom-events-and-interaction/tutorial.md). You will create named controls, use browser validation, add a cross-field rule, and explain the boundary between helpful frontend checks and trusted server validation.

## Form anatomy

A form gathers successful controls into a submission. `name` determines a submitted key; `id` connects labels and other document relationships. They serve different purposes. A visible `label` provides a persistent prompt; a placeholder is only a temporary hint. Use `fieldset` and `legend` for related controls such as a radio group. Supply appropriate `autocomplete` tokens so browsers and password managers can help users.

Input types influence keyboards, semantics, and built-in validation. `email` performs basic syntax checking, not mailbox verification. `number` suits numeric quantities; telephone numbers, postal codes, and account identifiers are usually strings because leading zeros and formatting matter. `inputmode` suggests a keyboard without defining a validation rule. `required`, `min`, `max`, `step`, `minlength`, `maxlength`, and `pattern` express different constraints. The [input reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input) documents which attributes apply to each type.

Unchecked checkboxes are normally absent from form data. Disabled controls are not submitted and cannot be focused; readonly text controls normally remain submitted. Repeated names can produce multiple values, so use `FormData.getAll()` for groups. File inputs produce `File` values rather than ordinary strings. For network submission, GET places fields in the URL and suits retrieval; POST places data in a request body and commonly creates or changes state. Neither method replaces HTTPS, authorization, CSRF defenses, or server-side validation.

## Worked tutorial: a reservation form

1. Create `index.html` and open it in a browser. This is a local demonstration: it displays the validated fields and does not contact a server.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Reserve a workshop place</title>
<main>
  <h1>Reserve a workshop place</h1>
  <form id="reservation">
    <p>
      <label for="email">Email address (required)</label>
      <input id="email" name="email" type="email"
        autocomplete="email" required>
    </p>
    <fieldset>
      <legend>Attendance dates</legend>
      <p>
        <label for="start">First day (required)</label>
        <input id="start" name="start" type="date" required>
      </p>
      <p>
        <label for="end">Last day (required)</label>
        <input id="end" name="end" type="date" required
          aria-describedby="date-help">
      </p>
      <p id="date-help">The last day must be on or after the first day.</p>
    </fieldset>
    <p>
      <label for="seats">Number of seats</label>
      <input id="seats" name="seats" type="number"
        min="1" max="4" step="1" value="1" required>
    </p>
    <button type="submit">Review reservation</button>
    <p id="status" role="status"></p>
  </form>
</main>
<script src="app.js" defer></script>
</html>
```

2. Save `app.js` beside it:

```javascript
const form = document.querySelector('#reservation');
const start = document.querySelector('#start');
const end = document.querySelector('#end');
const status = document.querySelector('#status');

function validateDates() {
  const invalid = start.value && end.value && end.value < start.value;
  end.setCustomValidity(invalid ? 'Choose a last day on or after the first day.' : '');
}

for (const control of [start, end]) {
  control.addEventListener('input', validateDates);
}

form.addEventListener('submit', (event) => {
  event.preventDefault();
  validateDates();
  if (!form.reportValidity()) return;
  const data = new FormData(form);
  status.textContent = `${data.get('seats')} seat(s), ${data.get('start')} to ` +
    `${data.get('end')}, contact ${data.get('email')}. Demo only; nothing was sent.`;
});
```

3. Submit empty fields, malformed email, zero seats, and an end date before the start. Fix each error, then submit valid values.

The expected result is browser-provided feedback for invalid fields and a review message for valid input. Normal date controls produce normalized date strings, so comparing complete `YYYY-MM-DD` strings works for this example's date-only values. This is not a general timezone-aware date comparison strategy.

`setCustomValidity('')` clears the custom error; forgetting it leaves the field invalid forever. Native interactive validation happens before a normal submit event, so cross-field rules are updated while editing. The submit handler also rechecks the rule before processing. A programmatic caller should use `requestSubmit()` for normal submission behavior; `submit()` bypasses the submit event and interactive validation.

## Error experience and production extensions

For larger forms, show persistent inline error text, associate it with the control, set `aria-invalid` when appropriate, and present a concise error summary linking to fields after a failed submission. Do not announce errors on every keystroke or rely on color alone. Preserve entered data after server failure. Browser validation is a useful starting point; internationalized, domain-specific rules and server errors often need a deliberate custom presentation. See [WAI form validation guidance](https://www.w3.org/WAI/tutorials/forms/validation/).

Client checks are bypassable. The server must validate types, ranges, permissions, availability, file content, and business rules again. Prevent duplicate writes through server-supported idempotency where needed. Treat loading, success, validation failure, network failure, and retry as distinct states. Do not permanently disable a submit button without explaining why or allowing recovery.

## Exercises and acceptance criteria

- Add an optional newsletter checkbox. Confirm its absence when unchecked and explicit value when checked.
- Add two workshop checkboxes with the same name. Read both with `getAll()` without losing a selection.
- Add persistent text explaining a date error. It must clear when corrected and remain associated with the end-date field.
- Write a server validation checklist for the reservation. Include a rule the frontend cannot verify, such as remaining capacity.

## References

- [MDN: constraint validation](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation)
- [MDN: FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [W3C WAI: form labels](https://www.w3.org/WAI/tutorials/forms/labels/)
- [WHATWG: forms](https://html.spec.whatwg.org/multipage/forms.html)
