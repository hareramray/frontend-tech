# End-to-end testing with Playwright

[Curriculum](../README.md)

Prerequisites: [component testing](../47-component-and-integration-testing/tutorial.md), [HTTP](../34-http-fetch-and-rest/tutorial.md), and [semantic HTML](../03-semantic-html/tutorial.md).

## Learning outcomes

Run a real browser against a local application, select controls through user-facing semantics, isolate test data, and diagnose failures using traces. End-to-end tests are most valuable for journeys where several boundaries must cooperate, such as sign-in, editing, checkout, and recovery from an interrupted save.

## Core concepts

A browser test exercises actual navigation, JavaScript execution, DOM behavior, and network requests. It is more realistic than a unit test but usually slower and more expensive to maintain. Keep a small set of meaningful critical paths and use lower-level tests for detailed rule combinations.

Playwright locators and assertions can wait for relevant conditions. That removes many arbitrary delays, but it does not make application readiness automatic: a page must expose a state that the test can observe. Independent tests should get independent browser contexts and isolated records. Shared mutable test accounts can create order-dependent failures. [Playwright installation](https://playwright.dev/docs/intro)

## Worked tutorial: test a complete form journey

1. Create an empty `e2e-lab` directory. Run:

```sh
npm init -y
npm install --save-dev @playwright/test http-server
npx playwright install chromium
```

2. Create `public/index.html`:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Lesson registration</title>
<main>
  <h1>Reserve a lesson</h1>
  <form>
    <label for="name">Your name</label>
    <input id="name" name="name" required>
    <button>Reserve</button>
  </form>
  <p role="status"></p>
</main>
<script>
  document.querySelector('form').addEventListener('submit', event => {
    event.preventDefault();
    const name = new FormData(event.currentTarget).get('name').trim();
    document.querySelector('[role="status"]').textContent =
      name ? `Reserved for ${name}.` : 'Enter your name.';
  });
</script>
</html>
```

3. Create `playwright.config.ts`:

```ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  use: {
    baseURL: 'http://127.0.0.1:8080',
    browserName: 'chromium',
    trace: 'retain-on-failure',
  },
  webServer: {
    command: 'npx http-server public -p 8080 -a 127.0.0.1 -c-1',
    url: 'http://127.0.0.1:8080',
    reuseExistingServer: !process.env.CI,
  },
});
```

4. Create `tests/reservation.spec.ts`:

```ts
import { test, expect } from '@playwright/test';

test('a learner can reserve a lesson', async ({ page }) => {
  await page.goto('/');
  await expect(page.getByRole('heading', { name: 'Reserve a lesson' })).toBeVisible();
  await page.getByLabel('Your name').fill('Asha');
  await page.getByRole('button', { name: 'Reserve', exact: true }).click();
  await expect(page.getByRole('status')).toHaveText('Reserved for Asha.');
});

test('an empty name does not produce a reservation', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Reserve', exact: true }).click();
  await expect(page.getByRole('status')).toBeEmpty();
  await expect(page.getByLabel('Your name')).toBeFocused();
});
```

5. Run `npx playwright test`. Expect two passing Chromium tests. `npx playwright test --headed` displays the browser; `npx playwright test --ui` opens an interactive runner. This sample verifies a browser journey, but its local confirmation does not create a real reservation on a server.

## Reliable tests in larger applications

Provision test records through supported APIs or fixtures, assign unique identifiers, and clean up only records the test owns. Prefer an isolated test environment for destructive flows. Authentication setup may reuse saved browser state, but that file can contain sensitive credentials: exclude it from version control and restrict its lifetime.

Network interception is useful for rare failures or unstable third parties. Be explicit about the scope of the claim: a test with every backend request mocked validates the frontend journey, not the deployed system's integration. Keep at least a focused set of genuine integration journeys where the risk justifies them.

Add browser projects for Firefox and WebKit when they are part of your support matrix. Browser emulation of a mobile viewport is not equivalent to testing every real device or operating system. Use real-device checks where keyboards, touch behavior, embedded webviews, or platform APIs are material risks.

## Pitfalls and debugging

Avoid `waitForTimeout` as a substitute for a useful readiness assertion. Avoid deep CSS paths that encode incidental markup. Do not make a test click through ten unrelated screens merely to reach a setup state an API can establish safely. Investigate flaky tests using their trace, network requests, console output, screenshots, and relevant server logs.

Retries may identify intermittent failures, but a suite that passes only through repeated retries has a reliability problem. Record the failure rate and fix shared-state races, resource limits, missing awaits, and inconsistent environments. Visual snapshots need controlled fonts, animations, viewport, and representative content.

## Exercises and acceptance criteria

- Add a whitespace-only name case. It must show a clear validation message and create no reservation.
- Add a real save endpoint in an isolated environment. The confirmation must correspond to a stored record, not only changed text.
- Introduce a failing assertion and inspect the retained trace. Identify the failed step without rerunning manually.
- Add a keyboard-only journey using Tab and Enter. Focus must remain visible and the same final outcome must be reached.

Primary references: [Playwright best practices](https://playwright.dev/docs/best-practices), [trace viewer](https://playwright.dev/docs/trace-viewer), and the installation guide above. Tooling documentation reviewed on 2026-10-18.
