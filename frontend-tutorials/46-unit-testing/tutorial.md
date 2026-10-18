# Unit testing: contracts, boundaries, and useful failures

[Curriculum](../README.md)

Prerequisites: [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md) and [promises](../18-promises-async-and-the-event-loop/tutorial.md). Continue with [component tests](../47-component-and-integration-testing/tutorial.md).

## Learning outcomes

Write tests around observable contracts, choose meaningful edge cases, isolate nondeterministic dependencies, and interpret coverage without confusing it with correctness. A unit test should make a business rule easier to change safely, not merely repeat implementation lines in another file.

## Core concepts

Arrange the inputs and environment, act through the public function or object, and assert the outcome. Keep a test's failure focused: a long chain of unrelated assertions makes diagnosis difficult. Boundary cases often reveal more than a large number of ordinary examples. For a quantity rule, try zero, the maximum, one beyond the maximum, negative values, fractional values, and non-numeric values.

Tests can run in Node.js, a DOM simulation, or a real browser. Choose the smallest environment that can represent the behavior. Pure arithmetic does not need a DOM. Layout, browser storage quirks, and focus behavior may require an actual browser. Vitest provides a runner, assertions, mocking, and coverage integrations; its environment and runtime requirements should match your project. [Vitest getting started](https://vitest.dev/guide/)

## Worked tutorial: a bounded quantity rule

1. Create an empty `unit-lab` directory and initialize it:

```sh
npm init -y
npm install --save-dev vitest
```

Use a supported Node.js release satisfying the installed Vitest requirements. Create `quantity.js`:

```js
export function changeQuantity(current, delta, maximum = 99) {
  if (![current, delta, maximum].every(Number.isSafeInteger)) {
    throw new TypeError('Quantities must be safe integers.');
  }
  if (maximum < 0 || current < 0 || current > maximum) {
    throw new RangeError('Current quantity is outside the allowed range.');
  }
  const next = current + delta;
  if (!Number.isSafeInteger(next)) throw new RangeError('Quantity overflow.');
  return Math.min(maximum, Math.max(0, next));
}
```

2. Create `quantity.test.js`:

```js
import { describe, expect, it } from 'vitest';
import { changeQuantity } from './quantity.js';

describe('changeQuantity', () => {
  it.each([
    [2, 1, 3],
    [2, -1, 1],
    [0, -1, 0],
    [99, 1, 99],
    [3, -100, 0],
  ])('changes %i by %i to %i', (current, delta, expected) => {
    expect(changeQuantity(current, delta)).toBe(expected);
  });

  it('honors a product-specific maximum', () => {
    expect(changeQuantity(4, 2, 5)).toBe(5);
  });

  it.each([NaN, Infinity, 1.5, '2'])('rejects invalid input %s', value => {
    expect(() => changeQuantity(value, 1)).toThrow(TypeError);
  });

  it('rejects inconsistent initial state', () => {
    expect(() => changeQuantity(10, 1, 5)).toThrow(RangeError);
  });
});
```

3. Run `npx vitest run`. Every case should pass. Run `npx vitest` for interactive watch mode while developing.
4. Temporarily remove the lower clamp from the implementation and rerun. The zero-boundary cases should fail. Restore the correct function. This deliberate mutation demonstrates that the tests detect a meaningful defect.

## Advanced techniques

Property-based tests can generate many combinations and verify invariants: valid results stay between zero and maximum, and zero delta preserves valid state. Such tests complement concrete examples; they do not replace a well-defined rule. If the product should reject rather than clamp an excessive quantity, update the contract and tests together.

Inject clocks, random generators, and service interfaces when they materially control behavior. Fake timers are useful for retry or debounce logic, but restore them after each test and flush the appropriate task queues. An asynchronous test must await its promise or return it; otherwise the test may finish before the assertion runs.

Mock external boundaries selectively. Mocking every internal function freezes the implementation structure and can make a refactor fail despite unchanged behavior. Conversely, a unit test that calls a live external API is slow, unreliable, and no longer isolated. Put integration evidence in a separate test layer.

## Coverage and debugging

Statement coverage answers which statements executed. Branch coverage exposes unvisited alternatives. Neither proves that assertions would detect a wrong result. Review tests around high-risk rules, error paths, and previous regressions. Snapshot tests are useful for stable structured output when small enough to review; huge snapshots can become unreadable approval rituals.

When a test fails, inspect the smallest input that reproduces the mismatch. Check shared mutable fixtures, leaked mocks, locale/time-zone dependence, and test order. A test that passes alone but fails in the suite often reveals isolation problems. Repeated retries should not be the permanent repair for a deterministic unit test.

## Exercises and acceptance criteria

- Add coverage for invalid `delta`, invalid `maximum`, and overflow. Each test must state the intended rule in its name.
- Add an invariant test over a bounded grid of inputs. Every result must remain within the supplied maximum.
- Create a separate pure formatter that displays zero as “None selected.” Test the wording without involving DOM rendering.
- Write one example of a behavior that belongs in a browser test instead, and explain what Node.js cannot establish.

Primary references: [Vitest guide](https://vitest.dev/guide/), [Vitest mocking](https://vitest.dev/guide/mocking), and [Vitest coverage](https://vitest.dev/guide/coverage). Runner setup was checked against official documentation on 2026-10-18.
