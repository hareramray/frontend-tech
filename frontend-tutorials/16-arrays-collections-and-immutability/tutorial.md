# Arrays, collections, and immutability

[Curriculum](../README.md)

## Prerequisites and outcomes

Complete [functions](../14-functions-scope-and-closures/tutorial.md) and [objects](../15-objects-prototypes-and-classes/tutorial.md). You will choose between arrays, Maps, and Sets, transform data without accidental mutation, preserve stable identity, and recognize shallow-copy boundaries.

## Choose a collection for the relationship

Arrays are ordered sequences indexed from zero. They are useful for rendering lists, maintaining order, and applying transformations. `map()` creates one output per visited input, `filter()` selects values, `find()` returns the first match or undefined, and `some()`/`every()` answer boolean questions. `reduce()` combines values through an accumulator; always consider an explicit initial value so empty input has defined behavior.

`push`, `pop`, `splice`, `sort`, and `reverse` mutate arrays. `slice`, spread, `map`, and `filter` create new arrays, while methods such as `toSorted()` and `toReversed()` provide copying alternatives in supporting runtimes. A copied array can still reference the same nested objects. `sort()` defaults to string-based comparison, so numeric sorting normally needs `(a, b) => a - b`. A comparator should be consistent and should not mutate the data it compares.

Map stores key-value relationships with keys of any type and preserves insertion order. It is useful for lookup by stable identifiers. Set stores unique values and also preserves insertion order. Object-valued keys and set members are compared by identity, not deep content. WeakMap and WeakSet hold weak references to object keys/members and do not provide ordinary enumeration; they suit auxiliary metadata whose lifetime should follow another object.

Immutability is a discipline of producing new values rather than changing existing ones. It makes previous states available for comparison, undo, and predictable rendering. It does not mean deep-cloning everything. Copy only the changed path and share unchanged values deliberately. Large immutable updates have allocation costs; normalize repeated entities by ID or use appropriate data structures when measurements justify it.

## Worked tutorial: an immutable task collection

1. Create `app.js` with this code. Run `node app.js`, or use the HTML wrapper in [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md).

```javascript
'use strict';

const original = [
  { id: 'html', title: 'Read HTML', done: true, minutes: 20 },
  { id: 'css', title: 'Practice Grid', done: false, minutes: 35 },
  { id: 'js', title: 'Trace a closure', done: false, minutes: 25 }
];

function toggleTask(tasks, id) {
  if (!tasks.some(task => task.id === id)) {
    throw new RangeError(`Unknown task: ${id}`);
  }
  return tasks.map(task => task.id === id
    ? { ...task, done: !task.done }
    : task);
}

const updated = toggleTask(original, 'css');
const remaining = updated.filter(task => !task.done);
const remainingMinutes = remaining.reduce((sum, task) => sum + task.minutes, 0);
const sorted = [...updated].sort((a, b) => a.minutes - b.minutes);
const byId = new Map(updated.map(task => [task.id, task]));
const uniqueTopics = [...new Set(['html', 'css', 'html', 'js'])];

console.log(remaining.map(task => task.title));
console.log(remainingMinutes);
console.log(sorted.map(task => task.id));
console.log(uniqueTopics);
console.assert(original[1].done === false);
console.assert(updated[1].done === true);
console.assert(original !== updated);
console.assert(original[0] === updated[0]);
console.assert(original[1] !== updated[1]);
console.assert(byId.get('css') === updated[1]);

const state = { tasks: original, preferences: { compact: false } };
const nextState = { ...state, tasks: updated };
console.assert(nextState.preferences === state.preferences);
```

2. Expect the remaining title to be `Trace a closure`, the remaining duration `25`, sorted IDs `html, js, css`, and unique topics `html, css, js`.
3. Inspect the identity assertions. The array and changed task are new objects; unchanged tasks retain their references.
4. Add another task with a unique ID and confirm lookup and sorting still work. Then experiment with a duplicate ID and observe that Map retains one value for that key.

The sorted copy leaves `updated` in its original order. In a supported runtime, `updated.toSorted((a, b) => a.minutes - b.minutes)` expresses the same copying intention more directly; consult [MDN's toSorted reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted) for compatibility. Stable IDs distinguish entity identity from its current list index, which changes during filtering and sorting.

Destructuring reads selected values into bindings, while rest syntax collects the remainder. Append this to the same file:

```javascript
const [firstTask, ...laterTasks] = updated;
const { title: displayTitle, done = false, ...taskMetadata } = firstTask;
console.log(displayTitle, done, laterTasks.length, taskMetadata.id);
```

It prints `Read HTML`, true, `2`, and `html`. The rename `title: displayTitle` changes the local variable name, not the object's key. Defaults apply when a value is undefined, not null. Object rest creates a new object containing remaining own enumerable properties; array rest creates a new array. Nested values still share references. Destructuring a null or undefined object throws, so validate uncertain boundary data instead of assuming the syntax makes it safe.

## Edge cases and tradeoffs

Array holes are missing properties, not ordinary explicit undefined entries. Different iteration methods treat holes differently, so avoid sparse arrays in normal UI data. Validate external arrays before transforming them: a missing `minutes` field can turn a total into NaN. Repeated `find()` calls across a large list can be wasteful; a Map may provide a clearer lookup structure, but building it also has a cost.

Spreading `state` does not clone `preferences`. Mutating `nextState.preferences.compact` would also affect `state.preferences`. To change that path, write `{ ...state, preferences: { ...state.preferences, compact: true } }`. `Object.freeze()` is shallow and is mainly a guard, not a replacement for a correct update strategy. JSON round trips lose or transform values and should not be a general deep-copy technique.

Typed arrays provide fixed-width numeric views over buffers for media, binary protocols, and performance-oriented data. They have different constraints from ordinary arrays. Iterators and generators allow lazy traversal without allocating an entire result upfront. Asynchronous iterators handle values arriving over time; cancellation and cleanup remain important when a consumer stops early.

Collection choice should follow required operations: ordered rendering, lookup, uniqueness, grouping, range queries, or binary access. Document whether callers may mutate returned data, whether order is meaningful, and what happens when an ID is missing. An explicit contract prevents subtle state bugs across components.

## Exercises and acceptance criteria

- Add an immutable rename operation. Only the selected task and containing array should receive new identities.
- Group tasks into completed and remaining arrays with one pass. Empty input must produce two empty arrays.
- Reject duplicate task IDs before building the Map. The error must identify the duplicate key.
- Add an undo stack of prior arrays. Undo must restore earlier values without later changes having modified them.

## References

- [MDN: Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
- [MDN: Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)
- [MDN: Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)
- [MDN: iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators)
