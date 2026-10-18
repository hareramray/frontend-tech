# Promises, async functions, and the event loop

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [functions and closures](../14-functions-scope-and-closures/tutorial.md) and [DOM events](../17-dom-events-and-interaction/tutorial.md). You will predict basic asynchronous ordering, handle success and failure, cancel obsolete work, and prevent older results from overwriting newer user intent.

## Understand scheduling before adding await

A JavaScript agent executes a running job to completion. The browser coordinates tasks such as user input and timers with microtasks such as promise reactions. At microtask checkpoints it drains the microtask queue, including newly added microtasks, before moving on. Rendering opportunities depend on the browser's event loop; a zero-delay timer is not a guarantee of an immediate callback or a paint between every pair of tasks. See [MDN's execution model](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Execution_model).

A Promise represents eventual fulfillment or rejection. Its executor runs synchronously; attached reaction callbacks run asynchronously. `then()` returns a new promise, so returning a value, returning another promise, or throwing changes the next step in the chain. `async` functions always return promises. `await` suspends the current async function, not the entire browser and not any CPU-heavy work you placed before the suspension.

Catch errors where you can recover or present useful feedback. A `try` around an unawaited asynchronous operation does not catch that operation's later rejection. Use `finally` for cleanup, but avoid returning from it because that can replace an earlier result or error. An unhandled rejection is a missed failure path, not a harmless logging detail.

Sequential awaits are appropriate when work depends on earlier results. `Promise.all()` waits for all fulfillments and rejects when one input rejects, without automatically canceling the others. `allSettled()` records every outcome, `race()` adopts the first settlement, and `any()` adopts the first fulfillment or rejects if all reject. Large workloads may need a concurrency limit rather than starting thousands of requests at once.

## Worked tutorial: cancel obsolete loading

1. Create `index.html` and `app.js`. Open the HTML in a browser. The loader below simulates asynchronous work locally, so no service or API key is required.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Async loading lab</title>
<main>
  <h1>Choose a practice profile</h1>
  <button type="button" id="slow">Load slow profile</button>
  <button type="button" id="fast">Load fast profile</button>
  <button type="button" id="cancel">Cancel loading</button>
  <p id="status" role="status">Choose a profile.</p>
</main>
<script src="app.js" defer></script>
</html>
```

```javascript
console.log('A: synchronous start');
setTimeout(() => console.log('D: timer task'), 0);
Promise.resolve().then(() => console.log('C: promise reaction'));
console.log('B: synchronous end');

function simulatedLoad(name, delay, signal) {
  return new Promise((resolve, reject) => {
    if (signal.aborted) {
      reject(new DOMException('Cancelled', 'AbortError'));
      return;
    }
    const timer = setTimeout(() => {
      signal.removeEventListener('abort', onAbort);
      resolve({ name, lessons: 4 });
    }, delay);
    function onAbort() {
      clearTimeout(timer);
      signal.removeEventListener('abort', onAbort);
      reject(new DOMException('Cancelled', 'AbortError'));
    }
    signal.addEventListener('abort', onAbort, { once: true });
  });
}

const status = document.querySelector('#status');
let active = null;

async function showProfile(name, delay) {
  active?.abort();
  const controller = new AbortController();
  active = controller;
  status.textContent = `Loading ${name}...`;
  try {
    const profile = await simulatedLoad(name, delay, controller.signal);
    if (active !== controller) return;
    status.textContent = `${profile.name}: ${profile.lessons} lessons.`;
  } catch (error) {
    if (active !== controller) return;
    status.textContent = error.name === 'AbortError'
      ? 'Loading cancelled.'
      : 'Unable to load the profile. Try again.';
  } finally {
    if (active === controller) active = null;
  }
}

document.querySelector('#slow').addEventListener('click', () => showProfile('Slow learner', 1200));
document.querySelector('#fast').addEventListener('click', () => showProfile('Fast learner', 200));
document.querySelector('#cancel').addEventListener('click', () => active?.abort());
```

2. Inspect Console: the ordering should be A, B, C, D. The promise reaction waits for synchronous code, then runs before the timer task in this example.
3. Click Slow and immediately Fast. The final result must describe Fast, even though Slow was started first.
4. Click Slow and then Cancel. The status must report cancellation without an unhandled rejection.

The AbortController communicates cancellation to a cooperating operation. It does not magically stop arbitrary code. The identity guard independently prevents stale results and stale cleanup from affecting a newer operation. Timer and listener cleanup avoid retaining unnecessary work. These patterns apply to actual `fetch()` requests as well as this local simulation.

## Production pitfalls and deeper scheduling

Fetch rejects for network-level failure or abort, but an HTTP error response such as 404 usually fulfills with a Response; inspect `response.ok` or status before using the body. Parsing can also fail, and cancellation can happen while reading it. Retry only failures that make sense to retry, use backoff with jitter, and consider whether repeating a write is safe.

`array.forEach(async item => ...)` does not await its callbacks. Use `for...of` with await for sequential processing, or map to promises and await an appropriate combinator for concurrent processing. Racing a timeout promise does not by itself cancel the underlying request; wire cancellation separately.

Long synchronous work blocks interaction even inside an async function. An endless chain of microtasks can also starve other work. Split work into meaningful chunks, yield through suitable scheduling mechanisms, or use a worker for CPU-intensive processing. Workers have separate execution agents and communicate through messages; they do not directly manipulate the page DOM.

## Exercises and acceptance criteria

- Add a simulated failure path. It must show a recoverable error message and permit a successful retry.
- Load two independent records with `Promise.all()`. Explain what happens to the second operation when the first rejects.
- Add a timeout that aborts the current operation. Both the timer and abort listener must be cleaned up after every outcome.
- Remove cancellation temporarily while retaining the identity guard. Repeated Slow/Fast interactions must still never display stale results.

## References

- [MDN: using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)
- [MDN: async functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)
- [MDN: AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)
- [WHATWG: event loops](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)
