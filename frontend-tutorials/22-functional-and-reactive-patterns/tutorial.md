# 22. Functional and reactive patterns

[Curriculum](../README.md)

Prerequisites: [data structures](../21-data-structures-and-algorithms/tutorial.md), promises, closures, and event listeners.

## What you will learn

Separate pure transformations from effects, model changes as events, and manage subscription lifetimes. Build a tiny observable store to understand the mechanics behind larger state and reactive libraries.

## Concepts and decisions

A pure function returns the same result for the same inputs and does not change external state. Purity makes local reasoning and testing easier: a price calculation can run without a document, network connection, or clock. Effects include rendering, storage, logging, network requests, and subscriptions. Place effects at explicit boundaries and keep business decisions in ordinary functions.

Immutability means creating a new representation of changed state instead of altering an existing shared value. It supports history, reference comparisons, and predictable updates. It does not require cloning every nested object. Structural sharing reuses unchanged branches while replacing changed ones. Spreading an object is shallow, so nested arrays still need deliberate treatment.

Function composition connects small transformations. Currying and partial application can make configuration reusable, but multiple layers of anonymous functions can reduce readability. Use `map` for transformation, `filter` for selection, and `reduce` for accumulation; a clear loop is also valid. Functional programming is a set of useful constraints, not a requirement to eliminate every variable.

Reactive programming represents changing values or event sequences and updates their consumers. Signals usually expose a current value and track dependencies. Observables can represent zero, one, or many asynchronous emissions, errors, and completion. A promise represents one eventual outcome. These abstractions differ in cancellation, timing, replay, and ownership. RxJS documents observable contracts in its [observable guide](https://rxjs.dev/guide/observable).

## Worked tutorial: a store with explicit cleanup

Use Node.js 22.12 or newer. Create `store.mjs` in an empty directory. No packages are required:

```js
function reducer(state, event) {
  switch (event.type) {
    case 'add':
      if (!event.title.trim()) return state;
      return { ...state, titles: [...state.titles, event.title.trim()] };
    case 'clear':
      return { ...state, titles: [] };
    default:
      return state;
  }
}

function createStore(reduce, initialState) {
  let state = initialState;
  const listeners = new Set();
  return {
    getState: () => state,
    dispatch(event) {
      const next = reduce(state, event);
      if (Object.is(next, state)) return;
      state = next;
      for (const listener of [...listeners]) listener(state);
    },
    subscribe(listener) {
      listeners.add(listener);
      listener(state);
      return () => listeners.delete(listener);
    }
  };
}

const store = createStore(reducer, { titles: [] });
const stop = store.subscribe(state => console.log(state.titles.length));
const before = store.getState();
store.dispatch({ type: 'add', title: '  Learn composition  ' });
store.dispatch({ type: 'add', title: ' ' });
console.log('Old snapshot:', before.titles.length);
stop();
store.dispatch({ type: 'clear' });
console.log('Current:', store.getState().titles.length);
```

Run `node store.mjs`. Expect initial `0`, then `1`, then an old snapshot count of `0` and current count of `0`. The empty title preserves the original reference and therefore avoids a notification. Unsubscribing prevents subsequent observer output. The example is an educational store: production requirements such as nested dispatch, listener exceptions, batching, and integration with concurrent rendering need an explicit design.

## Applying the pattern to asynchronous interfaces

An autocomplete combines input events, delay, a request, and a rendered result. Debounce waits for a quiet period; throttle limits frequency during continuous activity. Switching to a newer request should cancel or ignore the old request so stale data cannot replace recent results. Combining streams introduces ordering choices: run requests concurrently, sequentially, or replace prior work. Pick that behavior from the product requirement, not from the shortest operator name.

Derived values should usually be calculated from source state. Storing both a task list and a task count creates an invariant every mutation must preserve. A selector computes the count directly. Memoization is worthwhile when the calculation is expensive and inputs retain meaningful identity; it adds cache invalidation complexity otherwise.

## Pitfalls and debugging

Every subscription needs an owner and a disposal moment. A component that subscribes on every render can multiply work and leak memory. Effects that update their own dependencies can loop indefinitely. Reading a value once and closing over it may capture a stale snapshot; decide whether an operation needs that snapshot or the current value.

Trace events with sequence numbers, then inspect reducer output and the effect triggered by each transition. Keep side effects out of reducers so replay stays meaningful. Do not place credentials or sensitive payloads in a persisted event log. Document whether subscriptions emit immediately, since the initial emission changes initialization behavior.

## Exercises and acceptance criteria

1. Add a pure selector for titles beginning with a chosen letter. Acceptance: source state and previous results remain unchanged.
2. Add history and undo. Acceptance: two additions followed by undo restore one title, and subsequent additions do not modify prior snapshots.
3. Connect a browser input to a 250 ms debounce. Acceptance: rapid typing produces one search, teardown removes the listener and timer, and older results cannot overwrite newer ones.

## References

- [RxJS: operators](https://rxjs.dev/guide/operators)
- [Redux: prerequisites and reducer concepts](https://redux.js.org/usage/structuring-reducers/prerequisite-concepts)
- [MDN: closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures)
