# Accessibility testing: automation and human evaluation

[Curriculum](../README.md)

Prerequisites: [web accessibility](../05-web-accessibility/tutorial.md), [end-to-end testing](../48-end-to-end-testing/tutorial.md), and [complex interface patterns](../57-complex-accessible-interface-patterns/tutorial.md) when testing widgets.

## Learning outcomes

Combine automated checks with keyboard and assistive-technology evaluation, test meaningful UI states, and report defects in reproducible terms. Accessibility is part of functionality: an unusable form remains broken even when its visual design matches a reference.

## Core concepts

Automated tools can identify many structural problems, such as certain missing accessible names, invalid ARIA relationships, and some contrast failures. They cannot determine whether every alternative text is useful, a workflow is understandable, or focus lands in the most sensible place. W3C explicitly recommends human evaluation in addition to tools. [WAI evaluation overview](https://www.w3.org/WAI/test-evaluate/)

Evaluate pages as sequences of states. A closed menu, open menu, validation error, empty result, loading view, and confirmation can have different problems. A successful initial-page scan does not cover content revealed later. Include keyboard-only use, zoom and reflow, reduced motion, high-contrast or forced-color preferences, touch targets, screen-reader navigation, and alternative input where relevant.

## Worked tutorial: add accessibility checks to a browser test

1. Complete the local Playwright lab from [chapter 48](../48-end-to-end-testing/tutorial.md). In that project, install the axe integration:

```sh
npm install --save-dev @axe-core/playwright
```

2. Create `tests/accessibility.spec.ts`:

```ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('registration has no automatically detected violations', async ({ page }) => {
  await page.goto('/');
  const initial = await new AxeBuilder({ page }).analyze();
  expect(initial.violations).toEqual([]);

  await page.getByLabel('Your name').fill('Asha');
  await page.getByRole('button', { name: 'Reserve', exact: true }).click();
  await expect(page.getByRole('status')).toHaveText('Reserved for Asha.');
  const confirmed = await new AxeBuilder({ page }).analyze();
  expect(confirmed.violations).toEqual([]);
});

test('the form can be completed from the keyboard', async ({ page }) => {
  await page.goto('/');
  await page.keyboard.press('Tab');
  await expect(page.getByLabel('Your name')).toBeFocused();
  await page.keyboard.type('Asha');
  await page.keyboard.press('Tab');
  await expect(page.getByRole('button', { name: 'Reserve', exact: true })).toBeFocused();
  await page.keyboard.press('Enter');
  await expect(page.getByRole('status')).toHaveText('Reserved for Asha.');
});
```

3. Run `npx playwright test tests/accessibility.spec.ts`. Expect both tests to pass. This checks the deliberately small example's known focus order and two rendered states. If your page adds navigation, update the keyboard journey to reflect its actual design rather than forcing this tab count.
4. Remove the input's `label` association temporarily and rerun. The test should expose the inaccessible control. Restore the label. Review violation impact, affected nodes, and suggested guidance before applying a fix. [Playwright accessibility testing](https://playwright.dev/docs/accessibility-testing)

## Manual evaluation procedure

Start with the keyboard and keep your hands away from the pointer. Confirm you can reach every interactive element, see focus, operate controls, escape temporary layers, and recover from errors. Check that focus is neither lost nor trapped unintentionally. A modal intentionally contains focus while open, but must offer a reliable way to close it.

Increase browser zoom and inspect a narrow viewport. Text must remain readable; required actions must not disappear beyond an inaccessible clipped region. Check long translations and enlarged text independently, because zoom alone does not reproduce every typography setting. Test reduced motion without removing essential state feedback.

Use a screen reader with a browser combination relevant to the product's users. Navigate by landmarks, headings, links, and form controls. Listen for accessible names, instructions, state changes, validation errors, and status announcements. Avoid announcing every keystroke in a noisy live region. Include people with disabilities in usability evaluation where possible; tool expertise is not a substitute for lived experience.

## Reporting, advanced practice, and pitfalls

A useful defect report includes the page and state, browser/assistive-technology combination, exact steps, expected experience, actual behavior, and relevant criterion or pattern. Capture the minimum evidence needed without including personal data. Group repeated root causes so a component-level repair can address several screens.

Do not silence rules globally because a vendor widget fails. Determine whether the issue is a false positive, an application defect, or a dependency problem; track narrow, justified exceptions with owners. An “accessibility overlay” cannot guarantee that the underlying application is usable. Adding ARIA to a broken custom control is not automatically better than using a native element.

Define a supported evaluation matrix and revisit it when the user population, browser support, or component system changes. Accessibility standards and applicable obligations have different roles; this chapter teaches technical evaluation, not a jurisdiction-specific legal conclusion.

## Exercises and acceptance criteria

- Audit the error state of a multi-field form. Every error must identify its field and explain correction, and keyboard focus must follow the intended recovery flow.
- Test a modal through open, Tab, Shift+Tab, Escape, and return focus. Record each observed result.
- Write one defect report that another developer can reproduce without a meeting.
- Demonstrate one issue that passes axe but fails human evaluation, such as an unhelpful image description, and document the reasoning.

Primary W3C and Playwright references above were reviewed on 2026-10-18. Passing this lab is evidence for specific checks, not a claim of complete conformance.
