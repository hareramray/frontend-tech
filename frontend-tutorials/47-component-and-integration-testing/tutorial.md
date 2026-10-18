# Component and integration testing

[Curriculum](../README.md)

Prerequisites: [DOM events](../17-dom-events-and-interaction/tutorial.md), [unit testing](../46-unit-testing/tutorial.md), and [HTTP requests](../34-http-fetch-and-rest/tutorial.md).

## Learning outcomes

Test components through accessible user interactions, connect rendering and asynchronous services in one test, distinguish mocks from real integration boundaries, and identify behavior that requires a real browser. Component tests focus on a rendered unit; integration tests verify cooperation among units such as a form, validation rule, and service adapter.

## Core concepts

Querying controls by role and accessible name tests the interface users encounter. A selector based on an internal CSS class can survive even when the control loses its label or semantics. Testing Library encourages user-observable queries, while `user-event` models interactions as sequences rather than a single synthetic event. [DOM Testing Library](https://testing-library.com/docs/dom-testing-library/intro/), [user-event](https://testing-library.com/docs/user-event/intro/)

DOM simulations are useful but do not implement the full browser's layout, rendering, or accessibility stack. Passing a simulated-DOM test does not establish visual correctness or screen-reader usability. Keep network contracts explicit: a service fixture can exercise loading and error states but cannot prove the live server's response schema or authorization.

## Worked tutorial: load a greeting

1. Create an empty `component-lab` directory, then install the tools:

```sh
npm init -y
npm install --save-dev vitest jsdom @testing-library/dom @testing-library/user-event
```

Use a Node.js release supported by these packages. Create `greeting.js`:

```js
export function mountGreeting(root, loadGreeting) {
  root.innerHTML = '<button type="button">Load greeting</button><p role="status"></p>';
  // The HTML above is a fixed developer-owned template, not user input.
  const button = root.querySelector('button');
  const status = root.querySelector('[role="status"]');
  let mounted = true;

  async function load() {
    button.disabled = true;
    status.textContent = 'Loading…';
    try {
      const greeting = await loadGreeting();
      if (mounted) status.textContent = greeting;
    } catch {
      if (mounted) status.textContent = 'Could not load. Try again.';
    } finally {
      if (mounted) button.disabled = false;
    }
  }
  button.addEventListener('click', load);
  return () => {
    mounted = false;
    button.removeEventListener('click', load);
    root.replaceChildren();
  };
}
```

2. Create `greeting.test.js`:

```js
// @vitest-environment jsdom
import { afterEach, expect, it, vi } from 'vitest';
import { screen, waitFor } from '@testing-library/dom';
import userEvent from '@testing-library/user-event';
import { mountGreeting } from './greeting.js';

let dispose;
afterEach(() => {
  dispose?.();
  document.body.replaceChildren();
});

it('loads and announces the service response', async () => {
  const service = vi.fn().mockResolvedValue('Welcome, learner.');
  dispose = mountGreeting(document.body, service);
  const user = userEvent.setup();
  await user.click(screen.getByRole('button', { name: 'Load greeting' }));
  await waitFor(() => {
    expect(screen.getByRole('status').textContent).toBe('Welcome, learner.');
  });
  expect(service).toHaveBeenCalledTimes(1);
});

it('makes a failed request retryable', async () => {
  const service = vi.fn().mockRejectedValue(new Error('offline'));
  dispose = mountGreeting(document.body, service);
  const user = userEvent.setup();
  const button = screen.getByRole('button', { name: 'Load greeting' });
  await user.click(button);
  await waitFor(() => expect(screen.getByRole('status').textContent)
    .toBe('Could not load. Try again.'));
  expect(button.disabled).toBe(false);
});
```

3. Run `npx vitest run`. Expect two passing tests. Change the button's label without updating the test: its role query should fail with an accessible DOM description. Restore it afterward.
4. Add a deferred promise fixture to hold the request pending. Confirm the button is disabled until resolution. An immediately resolved promise is insufficient evidence for the visible loading state.

## Integration depth and advanced practice

The example integrates DOM rendering with an injected service. To test a real fetch adapter, intercept the network boundary with a tool such as Mock Service Worker and provide realistic success, malformed response, slow response, and server-error cases. Keep a separate contract test or staging check against the actual API schema. Network interception cannot establish that production authentication is correct.

For framework components, use the framework-specific Testing Library renderer or browser component-test support. Include required providers in a small, explicit test helper. Avoid a massive universal provider wrapper that hides which dependencies a component actually requires.

Test race conditions by resolving requests out of order. Verify stale results are ignored, cancellation does not produce a misleading failure message, and unmounted components do not update removed views. The mounted flag here suppresses rendering after cleanup; it does not cancel the underlying service operation.

## Pitfalls, debugging, and exercises

Await interactions and asynchronous assertions. Avoid arbitrary sleeps, assertions inside unawaited callbacks, and queries that silently accept several matching controls. Reset DOM, mocks, timers, and service handlers between tests. Do not assert internal state variables when the user-visible effect expresses the same contract.

- Add a retry that succeeds after an initial failure. The same button must recover and show the final greeting.
- Unmount while a deferred request is pending. Resolving it must not recreate content or throw.
- Write a browser test for keyboard focus and real layout. Explain why the jsdom result alone cannot prove either.
- Test the service adapter's handling of a non-2xx response. It must not announce an error response body as a successful greeting.

Primary references above were reviewed on 2026-10-18; see also [Vitest environments](https://vitest.dev/guide/environment) for runtime selection.
