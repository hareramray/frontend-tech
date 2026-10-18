# 33. Client state management

[Curriculum](../README.md)

Prerequisites: [functional and reactive patterns](../22-functional-and-reactive-patterns/tutorial.md) and [data structures](../21-data-structures-and-algorithms/tutorial.md).

## What you will learn

Classify state by ownership and lifetime, derive redundant values, and model workflows with explicit transitions. Build a checkout state machine that prevents impossible combinations of status flags.

## Concepts and decisions

Client state includes local input values, open menus, selected records, drafts, preferences, and workflow progress. Server state is a local view of data owned elsewhere and has freshness, caching, synchronization, and failure concerns. URL state describes shareable navigation. Keeping these categories distinct helps avoid copying every value into one global store.

Start with the smallest owner that covers all consumers. Lift state when siblings need it. Use a shared store when distant features require coordinated access and explicit update rules. A global store creates coupling and lifetime obligations; it should solve a demonstrated coordination problem. Context, dependency injection, and stores distribute state in different ways but do not remove the need to define ownership.

Derived values should normally be calculated from canonical data. A selected-ID set plus an entity map can describe selection without duplicating entire objects. Normalization is useful when many views share the same records, but it adds relationship maintenance. Immutable updates help preserve snapshots and make equality checks meaningful.

A collection of booleans can permit contradictory states such as `isSaving`, `isSaved`, and `hasError` all being true. A finite-state model represents one status at a time and defines legal events for each status. Larger workflows may need hierarchical or parallel state machines, but a small transition function often provides enough structure.

## Worked tutorial: explicit checkout transitions

Use Node.js 22.12 or newer. Create `checkout.mjs` in an empty directory; no dependencies are required:

```js
function transition(state, event) {
  switch (state.status) {
    case 'editing':
      if (event.type === 'SUBMIT' && state.quantity > 0) {
        return { status: 'submitting', quantity: state.quantity };
      }
      if (event.type === 'SET_QUANTITY' && Number.isSafeInteger(event.quantity)
          && event.quantity >= 0) {
        return { ...state, quantity: event.quantity };
      }
      return state;
    case 'submitting':
      if (event.type === 'SUCCESS') {
        return { status: 'complete', orderId: event.orderId };
      }
      if (event.type === 'FAILURE') {
        return { status: 'failed', quantity: state.quantity, message: event.message };
      }
      return state;
    case 'failed':
      if (event.type === 'RETRY') {
        return { status: 'submitting', quantity: state.quantity };
      }
      if (event.type === 'EDIT') {
        return { status: 'editing', quantity: state.quantity };
      }
      return state;
    case 'complete':
      return state;
    default:
      throw new Error(`Unknown status: ${state.status}`);
  }
}

let state = { status: 'editing', quantity: 1 };
function dispatch(event) {
  state = transition(state, event);
  console.log(state);
}

dispatch({ type: 'SUBMIT' });
dispatch({ type: 'SET_QUANTITY', quantity: 99 });
dispatch({ type: 'FAILURE', message: 'Connection interrupted' });
dispatch({ type: 'RETRY' });
dispatch({ type: 'SUCCESS', orderId: 'demo-order-1' });
```

Run `node checkout.mjs`. Expect submitting, unchanged submitting, failed, submitting, and complete states. Quantity editing is ignored while submission is pending. Completion contains an order ID rather than stale editing fields. This code demonstrates state transitions only; it does not submit a real order or implement payment processing.

In a UI, render the button label, disabled state, error message, and confirmation from this one status. An effect layer would start the request on an authorized transition and dispatch success or failure. Keep network calls outside the transition function so the same event sequence can be replayed deterministically.

## Persistence and concurrency

Persist only state that should survive reload. A preference may fit local storage; a large offline draft may fit IndexedDB. Include a schema version and migration strategy. Storage can be unavailable or full, and old data can be malformed. Treat loading persisted values as an untrusted boundary and provide recovery.

Do not persist transient states such as `submitting` without a reconciliation plan. After a crash, the server may have completed a request even if the client never received the response. For consequential writes, use server-supported idempotency or operation lookup rather than assuming retry means a new action is safe. Multi-tab editing adds another conflict source; define whether updates merge, prompt, or use last-write-wins.

Selectors provide a stable read interface to a store. Actions describe intent, while reducers or transition functions enforce changes. Avoid exposing mutable internal collections to arbitrary consumers. When using a library, understand subscription granularity and whether a selector returns a stable reference before adding memoization.

## Debugging and exercises

Record the event sequence and inspect the first unexpected transition. A state inspector is most useful when action names communicate domain intent. Avoid logging private form values or credentials. Reset application state on account changes when data should not cross user boundaries.

1. Add cancellation while submitting. Acceptance: late success events from the canceled operation cannot complete a newer operation; use an operation ID.
2. Add a pure selector for `canSubmit`. Acceptance: the UI and validation share the same documented rule without storing another boolean.
3. Persist only the editing draft. Acceptance: invalid stored data falls back safely, a version mismatch is handled, and completed order data is not accidentally restored as a new draft.

## References

- [Redux: organizing state](https://redux.js.org/faq/organizing-state)
- [React: choosing the state structure](https://react.dev/learn/choosing-the-state-structure)
- [XState: state machines](https://stately.ai/docs/machines)
