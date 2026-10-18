# 38. Forms in applications

[Curriculum](../README.md)

Prerequisites: semantic HTML forms, [client state](../33-client-state-management/tutorial.md), and [HTTP requests](../34-http-fetch-and-rest/tutorial.md).

## What you will learn

Coordinate field values, validation, submission, asynchronous feedback, and focus. Build an accessible contact form with native constraints and a simulated server-validation failure.

## Concepts and decisions

A form is a workflow: understand the request, enter information, correct mistakes, submit, and receive a clear result. Field state may include value, touched status, dirty status, validation result, and pending asynchronous validation. Form state may include submission status and errors not attributable to a single field. Avoid mixing all these concerns into one boolean.

Client validation improves feedback but cannot enforce trust. The server must validate and authorize the submitted data independently. Native constraints such as `required`, `type`, and `minlength` cover many basic rules. The Constraint Validation API exposes validity state and supports custom messages. Custom validity must be cleared when the condition is corrected. See [MDN's constraint-validation guide](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Constraint_validation).

Use visible labels and describe requirements before an error occurs. Associate field errors through `aria-describedby`, mark invalid fields with `aria-invalid`, and direct focus to a meaningful correction point after an unsuccessful submission. Do not rely on color alone. Avoid announcing an error on every keystroke before the user has had a chance to complete the field.

## Worked tutorial: validated contact request

Use a modern browser. Create `index.html` and open it directly. No dependencies are required. Submission is simulated in memory; no email is sent and no information leaves the browser.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Contact form laboratory</title>
<style>
  body { font: 1rem system-ui; max-width: 36rem; margin: 2rem auto; padding: 1rem; }
  label, input, textarea { display: block; }
  input, textarea, button { font: inherit; margin-block: .5rem; }
  input, textarea { width: 100%; box-sizing: border-box; padding: .6rem; }
  [aria-invalid="true"] { border: 2px solid #a51625; }
  .error { color: #a51625; min-height: 1.4em; }
</style>
<h1>Contact request</h1>
<form id="contact" novalidate>
  <label for="email">Email address</label>
  <input id="email" name="email" type="email" autocomplete="email"
    required aria-describedby="email-error">
  <p class="error" id="email-error"></p>
  <label for="message">Message</label>
  <p id="message-hint">Use at least 10 nonblank characters.</p>
  <textarea id="message" name="message" required
    aria-describedby="message-hint message-error"></textarea>
  <p class="error" id="message-error"></p>
  <button id="submit" type="submit">Send request</button>
  <p id="status" role="status"></p>
</form>
<script>
  const form = document.querySelector('#contact');
  const email = document.querySelector('#email');
  const message = document.querySelector('#message');
  const button = document.querySelector('#submit');
  const status = document.querySelector('#status');
  const fields = [email, message];
  let submitting = false;

  function showError(field, text) {
    document.querySelector(`#${field.id}-error`).textContent = text;
    if (text) field.setAttribute('aria-invalid', 'true');
    else field.removeAttribute('aria-invalid');
  }

  function validate(field) {
    field.setCustomValidity('');
    if (field === message && field.value.replace(/\s/g, '').length < 10) {
      field.setCustomValidity('Enter at least 10 nonblank characters.');
    }
    const error = field.validity.valid ? '' : field.validationMessage;
    showError(field, error);
    return !error;
  }

  for (const field of fields) {
    field.addEventListener('input', () => {
      if (field.hasAttribute('aria-invalid')) validate(field);
    });
  }

  form.addEventListener('submit', async event => {
    event.preventDefault();
    if (submitting) return;
    const valid = fields.map(validate);
    const firstInvalid = fields.find((field, index) => !valid[index]);
    if (firstInvalid) { firstInvalid.focus(); return; }
    const payload = Object.fromEntries(new FormData(form));
    submitting = true;
    button.disabled = true;
    fields.forEach(field => { field.readOnly = true; });
    status.textContent = 'Sending…';
    try {
      await new Promise(resolve => setTimeout(resolve, 700));
      if (payload.email.toLowerCase() === 'blocked@example.com') {
        showError(email, 'Use another email address for this demonstration.');
        email.focus();
        status.textContent = 'Please correct the email address.';
        return;
      }
      status.textContent = 'Request accepted in this demonstration.';
      form.reset();
    } finally {
      submitting = false;
      button.disabled = false;
      fields.forEach(field => { field.readOnly = false; });
    }
  });
</script>
</html>
```

Submit empty fields and expect focus on Email with associated feedback. Enter a valid email and a short message; only the message should remain invalid. Use `blocked@example.com` with a valid message to exercise the simulated server error. Correct it and resubmit; successful submission clears the form. The readonly fields prevent a pending response from erasing edits made during that submission.

## Integrating real services and frameworks

Replace the simulated delay with a request that handles non-success HTTP responses, network failure, timeout, and structured server errors. Preserve values after failure. An ambiguous timeout may leave the operation completed on the server; duplicate prevention may need server-supported idempotency, not only a disabled button.

Controlled inputs keep values in framework state; uncontrolled inputs let the DOM retain them and read values through refs or `FormData`. Both can work. Choose based on synchronization, conditional fields, performance, and integration needs. Form libraries can coordinate complex nested arrays and validation, but still require accessible markup and a clear server-error policy.

## Pitfalls and exercises

Disabled fields are omitted from `FormData`; readonly textual fields are included. File inputs require special handling and server-side file checks. Password managers and autofill depend on meaningful names, labels, and autocomplete attributes. Locale-sensitive numbers and dates need explicit parsing rules rather than naive string conversion.

1. Add a subject field and an error summary linking to invalid controls. Acceptance: keyboard activation of a summary link focuses the correct field.
2. Add asynchronous availability checking. Acceptance: stale responses cannot mark a newer value invalid, and validation does not submit the form accidentally.
3. Add a real mocked failure response to the local API from chapter 34. Acceptance: field and general errors remain distinct, values survive failure, and successful submission is announced once.

## References

- [WAI: form validation](https://www.w3.org/WAI/tutorials/forms/validation/)
- [MDN: FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
- [MDN: autocomplete](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/autocomplete)
