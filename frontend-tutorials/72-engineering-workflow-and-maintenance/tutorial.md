# 72. Engineering workflow and maintenance

[Curriculum](../README.md)

**Prerequisites:** [Git](../02-development-environment-cli-and-git/tutorial.md), [testing](../46-unit-testing/tutorial.md), and [observability](../58-observability-analytics-and-feature-flags/tutorial.md).

**You will learn to:** turn a report into a reproducible change, review risk, document decisions, and maintain a frontend over time.

## Define the problem before the implementation

A useful task explains the user, triggering action, observed behavior, expected behavior, and constraints. Include accessibility, performance, localization, and data handling when they affect acceptance. “Make search better” is not testable; “typing a new query must never show results from an older query” is.

Before changing code, reproduce the issue with the smallest reliable input. Record environment and build information. Separate facts from hypotheses: a slow page might involve network latency, repeated rendering, an oversized image, or a main-thread calculation. Profile the relevant path rather than optimizing the most unfamiliar code.

## Tutorial: fix a sorting defect with a regression test

Create `catalog.mjs` with this intentionally incorrect function:

```js
export function sortByPrice(products) {
  return products.sort((a, b) => String(a.priceCents).localeCompare(String(b.priceCents)));
}
```

The report is: “A 1,000-cent product appears before a 200-cent product, and sorting changes another view.” Those are two observable defects: lexical comparison and mutation of shared input.

Create `catalog.test.mjs`:

```js
import test from 'node:test';
import assert from 'node:assert/strict';
import { sortByPrice } from './catalog.mjs';

test('orders by numeric price without changing the source array', () => {
  const input = [
    { id: 'expensive', priceCents: 1000 },
    { id: 'cheap', priceCents: 200 }
  ];
  const before = structuredClone(input);
  const actual = sortByPrice(input);
  assert.deepEqual(actual.map(item => item.id), ['cheap', 'expensive']);
  assert.deepEqual(input, before);
  assert.notStrictEqual(actual, input);
});
```

Run `node --test catalog.test.mjs`; the numeric-order assertion should fail. Replace the function with:

```js
export function sortByPrice(products) {
  return [...products].sort((a, b) => a.priceCents - b.priceCents);
}
```

Run the test again. It should pass. This repair assumes prices are already validated finite numbers in a known unit; validation belongs at the data boundary. The function makes a new array but does not deep-clone products, which is sufficient because it changes only ordering.

The test describes a meaningful behavior, not the exact implementation. An alternative non-mutating numeric sort should also pass. Node's built-in [test runner](https://nodejs.org/api/test.html) supports this small dependency-free exercise.

## Review and ship the change

Keep a focused diff, inspect it yourself, and write a review description that states the trigger, changed behavior, and evidence. Include screenshots for visual changes when they help, but also describe keyboard and responsive behavior. A screenshot cannot verify focus order, network races, or screen-reader announcements.

Reviewers should examine trust boundaries, state ownership, failure paths, dependency impact, and whether tests cover the reported defect. Avoid turning every naming preference into an architectural blocker. When a disagreement reflects a recurring design choice, document the decision and its tradeoffs rather than relying on chat history.

Release with observability appropriate to the risk. A feature flag can limit exposure, but code behind a flag still needs access control and cleanup. Define the success signal, failure signal, owner, and removal date for temporary instrumentation or migration code.

## Maintain deliberately

Schedule dependency updates in small groups, read migration notes, and test the application path that uses the dependency. Security audit results need interpretation: distinguish a runtime exposure from a development-only issue and apply supported fixes without blindly forcing incompatible versions. Commit lockfiles and keep runtime requirements documented.

Track technical debt as concrete costs and outcomes: “replace duplicated validators so five forms apply the same boundary rules” is more actionable than “clean up forms.” Deprecate old APIs with a migration path. Remove obsolete flags, unused assets, abandoned experiments, and stale documentation when evidence supports removal.

After an incident, record impact, timeline, contributing conditions, detection gaps, and corrective actions. Focus on system improvements instead of assigning blame. A useful action has an owner and a verifiable outcome, such as a contract test or an alert tied to user-visible failures.

**Exercise:** write a small pull-request description for the sorting fix and add one edge-case test. Acceptance criteria: the description explains both defects, the test would fail against the original implementation, assumptions about input types are documented, and the change does not expand into unrelated refactoring.

**References:** [Git documentation](https://git-scm.com/doc), [Google engineering code-review practices](https://google.github.io/eng-practices/review/), [npm audit](https://docs.npmjs.com/cli/v11/commands/npm-audit).
