# 19. Modules, errors, and debugging

[Curriculum](../README.md)

Prerequisites: JavaScript functions, objects, promises, and browser developer tools from the [curriculum](../README.md).

## What you will learn

Split code into modules with deliberate public interfaces, preserve useful error information, and debug a failure from evidence instead of random edits. You will build a small invoice calculation whose invalid inputs produce actionable diagnostics.

## Concepts and decisions

An ECMAScript module has its own scope. Named exports make the supported interface explicit; default exports are useful when a file has one principal value. Static imports participate in a dependency graph before execution. Dynamic `import()` returns a promise and can defer loading an optional feature. Browser module scripts use `type="module"`, run in strict mode, and resolve relative imports as URLs. Node's `.mjs` extension explicitly selects the same module syntax. See the [MDN module guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules).

Keep dependencies pointed toward small, stable domain functions. A tax calculator should not import its checkout page. Circular imports sometimes work, but initialization order can expose an uninitialized binding; extracting shared contracts is usually clearer. Avoid barrel files that hide expensive dependencies or create cycles.

Errors have different owners. Invalid user input needs nearby feedback. A disconnected service needs retry or an offline state. A broken invariant needs developer investigation. Throw `Error` objects, preserve a cause when adding context, and catch where you can recover or translate the failure. A catch that only hides the error makes both users and maintainers less informed.

## Worked tutorial: trace an invalid invoice

Use Node.js 22.12 or newer. Create an empty `invoice-lab` directory. There are no dependencies or installation commands. Create `invoice.mjs`:

```js
export function invoiceTotal(lines) {
  if (!Array.isArray(lines)) throw new TypeError('lines must be an array');
  return lines.reduce((total, line, index) => {
    const { priceCents, quantity } = line;
    if (!Number.isSafeInteger(priceCents) || priceCents < 0) {
      throw new RangeError(`Line ${index + 1}: invalid priceCents`);
    }
    if (!Number.isSafeInteger(quantity) || quantity < 1) {
      throw new RangeError(`Line ${index + 1}: invalid quantity`);
    }
    const next = total + priceCents * quantity;
    if (!Number.isSafeInteger(next)) throw new RangeError('Total overflow');
    return next;
  }, 0);
}
```

Create `main.mjs`:

```js
import { invoiceTotal } from './invoice.mjs';

function calculate(lines) {
  try {
    return invoiceTotal(lines);
  } catch (cause) {
    throw new Error('Invoice calculation failed', { cause });
  }
}

console.log(calculate([{ priceCents: 1250, quantity: 2 }]));
try {
  calculate([{ priceCents: 1250, quantity: '2' }]);
} catch (error) {
  console.error(error.message);
  console.error(error.cause?.message);
}
```

Run `node main.mjs`. Expect `2500`, followed by the contextual message and `Line 1: invalid quantity`. The string is intentionally invalid: silently coercing it would conceal a boundary validation mistake. Prices use integer minor units to avoid common binary floating-point surprises, but real financial applications must also define currencies, rounding rules, and permitted ranges.

Run `node --inspect-brk main.mjs` and connect through the browser's Node inspection interface. Set a breakpoint on the range check, resume, and inspect the call stack, `line`, and `typeof quantity`. Enable pause on caught exceptions to stop at the original throw even though the outer catch handles it. Change the string to a number and confirm that the same path now succeeds.

## A repeatable debugging workflow

First record the smallest input and sequence that reproduces the problem. State expected and observed behavior. Locate the first incorrect value rather than the final broken screen. Use conditional breakpoints for rare records, watch expressions for changing state, and the network panel for status, response body, timing, and initiator. Reduce one variable at a time and rerun the original case after changing code.

Source maps connect bundled code to authored files; verify that the deployed artifact and map belong to the same build. Logs should include operation identifiers and safe metadata, not access tokens or personal payloads. An asynchronous rejection requires a handler on its promise chain; a synchronous outer `try` cannot catch a later callback's throw. `finally` is for cleanup and should not return a value that masks an earlier result or exception.

## Pitfalls and tradeoffs

Do not replace a useful error with `throw new Error('failed')` without preserving its cause. Do not retry deterministic validation failures. Do not expose raw stack traces in customer-facing messages. Assertions document programmer expectations but do not replace input validation. A global error handler is useful for reporting unexpected failures, while local boundaries still own recovery.

## Exercises and acceptance criteria

1. Add a discount function in a separate module. Accept only integer percentages from 0 through 100; verify boundary values and rejection of `NaN`.
2. Load an optional formatter using dynamic import. The formatter must load only after an explicit action, and an import failure must produce readable feedback.
3. Create a circular import, observe the failure, and remove it by extracting shared data. Explain the final dependency direction in three sentences.

## References

- [MDN: Error and error causes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error)
- [Node.js: debugging guide](https://nodejs.org/en/learn/getting-started/debugging)
- [Chrome DevTools: JavaScript debugging](https://developer.chrome.com/docs/devtools/javascript)
