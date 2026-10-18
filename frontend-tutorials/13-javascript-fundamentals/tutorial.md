# JavaScript fundamentals

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [development environment](../02-development-environment-cli-and-git/tutorial.md). You need a browser console or an installed Node.js runtime. You will distinguish values and bindings, validate external input, use control flow, and write a small deterministic calculation with observable results.

## Values, bindings, and expressions

JavaScript has primitive values: undefined, null, booleans, numbers, bigints, strings, and symbols. Objects are reference values and include arrays and functions. `typeof null` is the historical string `"object"`; use an explicit null comparison rather than treating `typeof` as a complete type system. `NaN` is a number value representing an invalid numeric result, and `Number.isNaN()` checks it without coercion.

`const` prevents rebinding a variable; it does not freeze an object stored in that variable. `let` permits rebinding and is block scoped. Prefer these over function-scoped `var` in new code. Reading a lexical binding before initialization throws; declarations are not permission to access a value before it exists. Choose names that state units and meaning, such as `priceCents` rather than `p`.

Operators can convert types. `"2" + 3` concatenates into `"23"`, while `"2" * 3` produces `6`. Prefer deliberate parsing at boundaries and strict equality for ordinary comparisons. Truthiness is not the same as validity: `0`, `""`, `false`, null, undefined, and NaN are falsy, but an empty array is truthy. `??` falls back only for null or undefined; `||` falls back for every falsy value. Optional chaining avoids access errors on nullish values, but it should not silently conceal a required missing field.

Numbers use binary floating-point, so many decimal fractions are approximate. Integer minor units are useful for this small currency example, provided values stay within the safe integer range and rounding rules are explicit. Real pricing systems also need currency-specific scales, taxes, discounts, and server-authoritative calculations. BigInt supports arbitrary-size integers but cannot be freely mixed with Number arithmetic.

Strings are immutable and use UTF-16 code units. `.length` does not always count user-perceived characters. Locale-aware formatting and segmentation may need `Intl` APIs. Regular expressions help recognize patterns, but complicated validation should remain understandable and should avoid pathological backtracking on untrusted input.

## Worked tutorial: a validated quote calculation

1. Create `app.js` containing the code below.

```javascript
'use strict';

function parseQuantity(raw) {
  if (typeof raw !== 'string' || raw.trim() === '') {
    throw new TypeError('Quantity must be a nonempty string.');
  }
  const quantity = Number(raw);
  if (!Number.isSafeInteger(quantity) || quantity < 1 || quantity > 100) {
    throw new RangeError('Quantity must be a whole number from 1 to 100.');
  }
  return quantity;
}

function quote(priceCents, quantity, discountPercent = 0) {
  if (!Number.isSafeInteger(priceCents) || priceCents < 0) {
    throw new RangeError('Price must be a nonnegative safe integer.');
  }
  if (!Number.isSafeInteger(quantity) || quantity < 1 || quantity > 100) {
    throw new RangeError('Quantity must be a whole number from 1 to 100.');
  }
  if (!Number.isInteger(discountPercent) || discountPercent < 0 || discountPercent > 100) {
    throw new RangeError('Discount must be a whole percentage from 0 to 100.');
  }
  const subtotalCents = priceCents * quantity;
  if (!Number.isSafeInteger(subtotalCents) ||
      !Number.isSafeInteger(subtotalCents * discountPercent)) {
    throw new RangeError('This quote exceeds the supported calculation range.');
  }
  const discountCents = Math.round(subtotalCents * discountPercent / 100);
  return subtotalCents - discountCents;
}

const quantity = parseQuantity('3');
const totalCents = quote(1250, quantity, 10);
const formatter = new Intl.NumberFormat('en-US', {
  style: 'currency', currency: 'USD'
});
console.log(formatter.format(totalCents / 100));
console.assert(totalCents === 3375, 'Expected a 3375-cent total.');

for (const input of ['', '0', '2.5', 'many']) {
  try {
    parseQuantity(input);
    console.error('Unexpected accepted input:', input);
  } catch (error) {
    console.log(`Rejected ${JSON.stringify(input)}: ${error.message}`);
  }
}
```

2. Run it in a terminal with `node app.js`, or create `index.html` containing the following wrapper and open the browser console:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>JavaScript calculation lab</title>
<h1>Open developer tools to inspect the calculation.</h1>
<script src="app.js"></script>
</html>
```

3. Confirm the first output is `$33.75`, followed by four rejection messages. The assertion should not report a failure.

The parser validates a boundary value before the calculation uses it. `Number()` converts the complete string, unlike `parseInt()` which can accept a numeric prefix such as `"3items"`. This parser deliberately permits numeric forms such as `"3e0"`; if a product requires decimal digits only, add an explicit format rule before conversion. The quote rounds one aggregate discount to the nearest cent; a different business policy may round each line separately.

## Debugging and control flow

Use `if` for decisions, `switch` for explicit multiway cases, and loops appropriate to the data. `for...of` iterates values of an iterable, while `for...in` enumerates property keys and is usually the wrong choice for array values. `break`, `continue`, and early returns can simplify logic, but excessive nesting obscures invariants.

Throw meaningful errors when a function's contract cannot be met. Catch errors where you can recover or present useful context; swallowing every exception makes defects harder to locate. Developer-tool breakpoints, step execution, scope inspection, and stack traces explain the actual execution path more reliably than scattered logging alone.

## Dates, structured text, and patterns

Append this small experiment to `app.js` to inspect three common boundary tools:

```javascript
const reservation = JSON.parse('{"startsAt":"2027-01-16T09:00:00Z","seats":3}');
const startsAt = new Date(reservation.startsAt);
if (Number.isNaN(startsAt.getTime())) throw new TypeError('Invalid start time.');
const dateFormat = new Intl.DateTimeFormat('en-GB', {
  dateStyle: 'medium', timeStyle: 'short', timeZone: 'Asia/Kolkata'
});
console.log(dateFormat.format(startsAt));
console.log(JSON.stringify({ ...reservation, confirmed: false }));
console.assert(/^[0-9]+$/u.test('003'));
console.assert(!/^[0-9]+$/u.test('3items'));
```

The timestamp denotes an instant with an explicit UTC suffix; the formatter displays that instant in a chosen zone, around 14:30 on 16 January 2027. `Date` stores a timestamp, not a named timezone or a calendar-only birthday. Avoid locale-dependent date parsing and distinguish an instant from a date-only business value. JSON parses syntax but does not validate your domain schema; inspect required fields and types afterward. `JSON.stringify()` creates structured text with limitations explained in the objects chapter. The anchored regular expression accepts only ASCII digits. Adding `g` changes repeated `test()` behavior because the expression tracks `lastIndex`; do not use it accidentally for a simple repeated validator.

## Exercises and acceptance criteria

- Add explicit decimal-digit-only quantity input. `"03"` may be accepted, but `"3e0"` and `"3items"` must be rejected.
- Add cases for a zero discount and a full discount. Results must be 3750 and 0 cents for three 1250-cent items.
- Explain the different results of `0 || 10` and `0 ?? 10` using executable assertions.
- Create a boundary test for unsafe arithmetic. The function must reject it rather than silently return an imprecise integer.

## References

- [MDN: grammar and types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types)
- [MDN: equality comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness)
- [MDN: Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)
- [ECMAScript language specification](https://tc39.es/ecma262/)
- [MDN: Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
- [MDN: regular expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)
