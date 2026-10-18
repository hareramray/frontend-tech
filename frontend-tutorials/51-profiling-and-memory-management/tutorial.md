# Profiling and memory management

[Curriculum](../README.md)

Prerequisites: [DOM events](../17-dom-events-and-interaction/tutorial.md), JavaScript closures, and [performance metrics](../50-web-performance-and-core-web-vitals/tutorial.md).

## Learning outcomes

Profile a repeatable interaction, distinguish expensive work from retained memory, identify resource ownership, and pair acquisition with cleanup. JavaScript garbage collection reclaims unreachable objects; it cannot infer that a reachable event listener, timer, subscription, or cache entry is no longer useful.

## Core concepts

A performance profile records where time goes. Look at long tasks, call stacks, scripting, style calculation, layout, painting, and network dependencies. A function that appears frequently may be cheap per call but expensive in total; a single long task can block interaction even when average CPU utilization is low.

A memory leak is unwanted retention over time. Repeated navigation that leaves detached DOM trees reachable through listeners or stores is a common example. Memory bloat is excessive memory without necessarily unbounded growth; allocation churn repeatedly creates and discards objects, potentially increasing garbage-collection work. Heap snapshots and allocation recordings answer different questions. [Chrome memory diagnostics](https://developer.chrome.com/docs/devtools/memory-problems)

## Worked tutorial: mount and dispose a live panel

1. Create `memory-lab/index.html`. Open it in a modern browser; no dependency installation is needed.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Resource ownership lab</title>
<main>
  <h1>Live panel</h1>
  <button id="toggle">Open panel</button>
  <div id="host"></div>
</main>
<script type="module">
  const host = document.querySelector('#host');
  const toggle = document.querySelector('#toggle');
  let dispose = null;

  function mountPanel(host) {
    const panel = document.createElement('section');
    panel.setAttribute('aria-label', 'Viewport information');
    const text = document.createElement('p');
    panel.append(text);
    host.append(panel);
    const controller = new AbortController();
    let ticks = 0;
    let closed = false;

    function render() {
      text.textContent = `Width: ${innerWidth}; ticks: ${ticks}`;
    }
    window.addEventListener('resize', render, { signal: controller.signal });
    const interval = setInterval(() => { ticks += 1; render(); }, 1000);
    const observer = new ResizeObserver(() => {
      panel.dataset.width = String(Math.round(panel.getBoundingClientRect().width));
    });
    observer.observe(panel);
    render();

    return () => {
      if (closed) return;
      closed = true;
      controller.abort();
      clearInterval(interval);
      observer.disconnect();
      panel.remove();
    };
  }

  toggle.onclick = () => {
    if (dispose) {
      dispose();
      dispose = null;
      toggle.textContent = 'Open panel';
    } else {
      dispose = mountPanel(host);
      toggle.textContent = 'Close panel';
    }
  };
</script>
</html>
```

2. Open and close the panel twenty times. After closing, its interval, resize listener, and observer should no longer perform panel work. The cleanup function is idempotent: calling it twice does not create a new failure.
3. In DevTools Memory, take a baseline heap snapshot, repeat the interaction, close the panel, request garbage collection using the available DevTools control, and take another snapshot. Compare retained panel objects and inspect retainer paths. Do not infer a leak from one temporary allocation spike.
4. In a disposable copy of this lab, omit `clearInterval(interval)` and repeat. The timer closure can retain the panel's data after removal. Restore cleanup and compare the retention path. The purpose is to learn the ownership chain, not to rely on a particular absolute memory number.

## A disciplined profiling procedure

Write down the operation and expected scale before recording: opening a menu once, filtering ten thousand rows, or switching a route fifty times. Record a production-like build, preserve the trace, make one targeted change, and repeat under comparable conditions. Developer tools themselves can retain inspected objects; clear console references and avoid logging every object while measuring memory.

For CPU issues, start with the longest relevant task and follow its call stack to the first actionable application boundary. Read-before-write batching can reduce repeated layout. Memoization can trade computation for retained memory, but it needs bounded keys and invalidation. A worker can move CPU work off the main thread while introducing message-copying and synchronization costs.

## Advanced ownership and pitfalls

Clean up animation frames, subscriptions, sockets, media tracks, observers, object URLs, and third-party widgets according to their APIs. Aborting fetch stops supported work but does not automatically undo application state changes. A detached element is not inherently a leak if it is short-lived and becomes unreachable soon.

WeakMap is useful when metadata should not keep an object alive. It is not an automatic replacement for a cache requiring enumeration, expiry, or bounded capacity. Garbage-collection timing and finalizers are unsuitable for essential cleanup such as releasing a camera or ending a subscription.

In framework code, place acquisition and cleanup in the corresponding lifecycle mechanism. Test setup-cleanup-setup sequences; development modes may intentionally expose missing cleanup. Avoid attributing every growth pattern to the framework before tracing the actual retained object.

## Exercises and acceptance criteria

- Add an animation frame loop and cancel it on close. No frames should keep updating the removed panel.
- Add a bounded cache of at most twenty entries. A stress run must not increase its size beyond that limit.
- Capture a retainer path for the intentional timer leak and explain every link from root to panel.
- Profile a large-list filter and separate algorithm time from DOM update time. Report a measured before/after improvement, including the test conditions.

Primary references: [Chrome memory guidance](https://developer.chrome.com/docs/devtools/memory-problems), [Performance panel](https://developer.chrome.com/docs/devtools/performance), and [MDN memory management](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management). Official tooling guidance reviewed on 2026-10-18.
