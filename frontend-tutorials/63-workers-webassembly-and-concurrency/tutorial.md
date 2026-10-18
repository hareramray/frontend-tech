# 63. Workers, WebAssembly, and concurrency

[Curriculum](../README.md)

**Prerequisites:** [event loop](../18-promises-async-and-the-event-loop/tutorial.md), [modules](../19-modules-errors-and-debugging/tutorial.md), and [profiling](../51-profiling-and-memory-management/tutorial.md).

**You will learn to:** move CPU work away from interaction, transfer data safely, and decide when WebAssembly is useful.

## Separate asynchronous work from parallel work

An `async` function does not automatically move calculations to another thread. A long loop still occupies the thread that executes it. Workers create a separate execution context. A dedicated worker serves one owner; a shared worker can serve several browsing contexts where supported. A service worker is an event-driven network intermediary with a different lifecycle, not a continuously running background process.

Workers communicate with messages and cannot directly update the page DOM. Most values cross the boundary using structured cloning. Transferable objects can move ownership of an underlying resource instead of copying it. Once an `ArrayBuffer` is transferred, the sender must not continue treating it as an owned readable buffer. These mechanics and worker restrictions are covered in [Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers).

## Tutorial: calculate in a worker

Create `index.html`, `main.js`, and `sum-worker.js`. Use an HTML page containing `<button id="run">Calculate</button><p id="result" role="status"></p><script type="module" src="main.js"></script>` inside a normal document. Serve it through localhost; a file URL is unsuitable for this exercise.

`sum-worker.js`:

```js
self.onmessage = ({ data }) => {
  const numbers = new Float64Array(data.buffer);
  let sum = 0;
  for (const number of numbers) sum += number;
  self.postMessage({ id: data.id, sum });
};
```

`main.js`:

```js
const button = document.querySelector('#run');
const result = document.querySelector('#result');
let worker;
let requestId = 0;

button.addEventListener('click', () => {
  worker?.terminate();
  const currentWorker = new Worker(new URL('./sum-worker.js', import.meta.url), { type: 'module' });
  worker = currentWorker;
  const id = ++requestId;
  const numbers = new Float64Array(1000000).fill(2);
  result.textContent = 'Calculating…';
  currentWorker.onmessage = ({ data }) => {
    if (data.id !== requestId) return;
    result.textContent = `Sum: ${data.sum}`;
    currentWorker.terminate();
    worker = undefined;
  };
  currentWorker.onerror = () => {
    if (id !== requestId) return;
    result.textContent = 'Calculation failed.';
    currentWorker.terminate();
    worker = undefined;
  };
  currentWorker.postMessage({ id, buffer: numbers.buffer }, [numbers.buffer]);
  console.log(numbers.buffer.byteLength); // 0: transferred away
});
window.addEventListener('pagehide', () => worker?.terminate());
```

Expect `Sum: 2000000`. The input allocation still happens on the main thread; for truly large inputs, generate or parse them in the worker too. This small calculation may be slower overall because worker startup and messaging have overhead. Measure responsiveness and total completion time separately.

## Cancellation, scheduling, and shared memory

Terminating a worker immediately abandons its in-memory state. A cancellation message cannot interrupt a synchronous loop already running inside that worker; the loop must yield or consult a suitable shared cancellation flag. Reuse a worker or bounded pool for repeated work rather than creating one per keystroke. Attach request IDs so slow old results cannot replace newer output.

`SharedArrayBuffer` allows shared memory in eligible environments, commonly requiring cross-origin isolation. Shared memory introduces races; use `Atomics` and a defined protocol rather than assuming reads and writes arrive in the intended order. Cross-origin isolation headers can affect third-party resources and embedded content. It is an architectural choice, not a harmless performance toggle.

For work that must remain on the main thread, divide it into short chunks and yield between them. `requestAnimationFrame` schedules work near rendering; placing a huge loop inside it still blocks the frame. `requestIdleCallback`, where available, suits nonessential work with fallback and timeout policies; it should not be the only route to required completion.

## Where WebAssembly fits

WebAssembly is a portable compilation target commonly used for existing C/C++/Rust libraries, codecs, simulations, and compute-heavy processing. It does not automatically provide DOM access, eliminate memory use, or outperform JavaScript. JavaScript-to-Wasm crossings and data conversion have costs. Batch operations, profile realistic inputs, and retain clear error boundaries.

A compiled module can run in a worker. A typical integration uses `WebAssembly.instantiateStreaming(fetch('/module.wasm'), imports)` with correct MIME type and fallback handling, then calls its exported functions. The precise imports, memory layout, and cleanup depend on the compiler toolchain; do not invent a universal ABI. SIMD and threads need separate compatibility checks.

**Exercise:** add progress and cancel controls for processing a large text file. Acceptance criteria: typing in another input remains responsive, progress is monotonic, cancellation releases the worker, stale results are ignored, and a measured report compares cloning with transferring an equivalent buffer.

**References:** [Transferable objects](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Transferable_objects), [WebAssembly concepts](https://developer.mozilla.org/en-US/docs/WebAssembly/Guides/Concepts), [SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer).
