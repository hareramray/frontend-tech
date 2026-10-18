# Functions, scope, and closures

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md). You will define clear function contracts, reason about lexical scope, use closures to preserve private state, and avoid common callback and `this` mistakes.

## Functions are values with contracts

A function receives arguments, performs work, and returns a value or throws. A missing explicit return produces undefined. JavaScript does not enforce declared parameter counts, so defaults, validation, and clear documentation matter. Default parameters apply to undefined or an omitted argument, not to null. Rest parameters collect remaining arguments into an array; spread syntax expands iterable values at a call site. They look similar but serve opposite directions of data flow.

Function declarations, function expressions, arrow functions, and methods have different creation and binding behavior. A function declaration is available through declaration instantiation in its scope; a function expression assigned to a `const` cannot be read before initialization. Arrows capture lexical `this` and have no own `arguments`; ordinary function calls determine `this` from the call form. Use arrows for callbacks that should retain surrounding context, not as an automatic replacement for every object method.

Lexical scope means a function resolves variable names according to where it was defined, not where it was called. A closure is a function together with access to its surrounding bindings. Those bindings can remain available after the outer function returns. The closure observes current binding values rather than an automatic frozen copy of each value. This is why closures are useful for state and also why delayed callbacks can observe a later value.

Pure functions depend only on their inputs and avoid externally visible side effects. They are easy to test and reuse. Interfaces still need effects such as DOM updates and network calls; keep those effects near explicit boundaries. Composition combines small functions through their inputs and outputs. Avoid splitting code into tiny functions that obscure a simple flow without introducing meaningful concepts.

## Worked tutorial: isolated counters with private state

1. Create `app.js` with the following code. Run `node app.js`, or use the browser wrapper from [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md).

```javascript
'use strict';

function createCounter(initial = 0) {
  if (!Number.isSafeInteger(initial)) {
    throw new TypeError('Initial value must be a safe integer.');
  }
  let value = initial;

  function increment(step = 1) {
    if (!Number.isSafeInteger(step) || !Number.isSafeInteger(value + step)) {
      throw new RangeError('Step must produce a safe integer.');
    }
    value += step;
    return value;
  }

  function reset() {
    value = initial;
    return value;
  }

  return Object.freeze({
    increment,
    reset,
    read: () => value
  });
}

const first = createCounter(10);
const second = createCounter();
console.log(first.increment());
console.log(first.increment(4));
console.log(second.read());
console.log(first.reset());
console.assert(first.read() === 10);
console.assert(second.read() === 0);

function onceAfterSuccess(fn) {
  let completed = false;
  let result;
  return function (...args) {
    if (!completed) {
      result = fn.apply(this, args);
      completed = true;
    }
    return result;
  };
}

let calls = 0;
const initialize = onceAfterSuccess(() => {
  calls += 1;
  return { ready: true };
});
const a = initialize();
const b = initialize();
console.assert(a === b && calls === 1);
```

2. Expect the logged values `11`, `15`, `0`, and `10`. No assertion should fail.
3. Add `first.increment(-2)` and inspect the new value. Then call `second.increment()` and confirm it changes only the second counter.
4. Inspect a breakpoint inside `increment`. The scope panel should show the captured `value` and `initial` for that particular counter.

Each factory call creates a distinct lexical environment. Freezing the returned API prevents replacing its methods, while its private captured `value` still changes through those methods. Privacy here comes from access to the binding, not from freezing the internal state.

`onceAfterSuccess` demonstrates a higher-order function: it accepts a function and returns another function. It preserves the caller's `this` through `apply`. If `fn` throws synchronously, completion is not recorded and a later call can retry. This exact wrapper is designed for synchronous functions: a returned rejected promise would still count as a completed call. An asynchronous version needs an explicit policy for in-flight calls and failed results.

## Common closure and callback mistakes

Using one function-scoped `var` binding in a loop can make callbacks all read the final value. A `let` loop binding creates the per-iteration bindings normally intended. Capturing a large object in a long-lived listener can also keep it reachable; remove listeners and timers when their owner ends. Garbage collection follows reachability, not the textual end of a function.

Passing a method as a callback loses its original receiver unless it is bound or wrapped. `setTimeout(object.method, 0)` does not mean “call as a method on object.” Use `() => object.method()` or `object.method.bind(object)` when that receiver is required, and retain a stable reference if you must remove the callback later.

Debouncing waits for a quiet interval before running work; throttling limits execution frequency over time. They solve different interaction problems and need cleanup. Memoization caches results for inputs, but cache keys, mutation, memory growth, and invalidation are part of the contract. Recursive functions need a terminating case and should not assume every runtime eliminates tail calls.

## Exercises and acceptance criteria

- Add a `decrement()` operation without exposing the private binding. Counters must stay independent.
- Implement a cancellable debounce wrapper. Rapid calls should produce one final call; cancellation must prevent it.
- Write a callback example that loses `this`, then fix it using both a wrapper and `bind`.
- Build two memoized functions with separate caches. Describe the key strategy and demonstrate that changing one cache does not affect the other.

## References

- [MDN: functions guide](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- [MDN: closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
- [MDN: arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions)
- [MDN: this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this)
