# 21. Data structures and algorithms for interfaces

[Curriculum](../README.md)

Prerequisites: [modules](../19-modules-errors-and-debugging/tutorial.md), arrays, objects, functions, and loops.

## What you will learn

Choose data structures from the operations an interface performs, estimate their costs, and separate computational bottlenecks from rendering bottlenecks. Build a searchable product catalog with stable identity and predictable ordering.

## Concepts and decisions

An array represents an ordered sequence and supports iteration naturally. A `Map` associates keys with values without inheriting object property names. A `Set` records membership and removes duplicates. Plain objects work well for fixed records, while maps often communicate dynamic collections more clearly. Weak maps and weak sets do not keep object keys alive, making them useful for private metadata whose lifetime follows another object. They are intentionally not enumerable.

Big-O describes growth, not a stopwatch result. A single scan is O(n); a nested full scan is O(n²); comparison sorting is typically O(n log n). Actual costs also depend on allocation, string normalization, cache behavior, and DOM work. JavaScript specifies useful behavior for built-ins but does not promise every operation has an identical implementation across engines. Map access is required to be sublinear on average, rather than universally guaranteeing constant time. See [MDN's Map reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map).

A stack supports undo and depth-first traversal. A queue supports work scheduling and breadth-first traversal. Trees model menus and document hierarchies. Graphs model dependencies and relationships with cycles. A visited set prevents an infinite traversal of cyclic graphs. Binary search requires sorted data and a consistent comparator; it is inappropriate for an unsorted list merely because it sounds faster.

## Worked tutorial: index and search products

Use Node.js 22.12 or newer. In an empty directory create `catalog.mjs`; there are no dependencies:

```js
const products = [
  { id: 'a', name: 'Canvas Bag', price: 1800 },
  { id: 'b', name: 'Travel Mug', price: 1200 },
  { id: 'c', name: 'Canvas Pouch', price: 900 }
];

function buildCatalog(rows) {
  const byId = new Map();
  const searchable = [];
  for (const row of rows) {
    if (byId.has(row.id)) throw new Error(`Duplicate ID: ${row.id}`);
    const record = Object.freeze({ ...row });
    byId.set(record.id, record);
    searchable.push({ record, searchName: record.name.toLocaleLowerCase('en') });
  }
  return {
    get(id) { return byId.get(id); },
    search(query) {
      const needle = query.trim().toLocaleLowerCase('en');
      return searchable
        .filter(item => item.searchName.includes(needle))
        .map(item => item.record)
        .sort((a, b) => a.price - b.price || a.id.localeCompare(b.id));
    }
  };
}

const catalog = buildCatalog(products);
const selectedIds = new Set(['a', 'a', 'c']);
console.log(catalog.get('b').name);
console.log(catalog.search('canvas').map(product => product.id));
console.log(selectedIds.size);
```

Run `node catalog.mjs`. Expect `Travel Mug`, an array containing `c` then `a`, and `2`. The source data remains unchanged. The index is built once; repeated lookup avoids repeated scans. Search remains linear because arbitrary substring matching must inspect candidate names in this implementation. Sorting only matching results reduces work when the result set is small.

Now add a duplicate row and observe the explicit failure. Duplicate identity must be resolved at ingestion, because silently overwriting data can make selection and editing target the wrong record. Add a product with the same price and verify that the ID tie-breaker keeps ordering deterministic. For real multilingual search, define locale, accents, tokenization, and matching requirements instead of assuming lowercase substring matching covers every language.

## Scaling the design

An index costs memory and maintenance. If products update, either rebuild the catalog for a small collection or update every affected index consistently. Normalized application state can store records by ID and keep ordered ID arrays separately. This reduces duplicated entity data but requires care when deleting references.

Measure the full interaction with realistic data. A fast filter followed by 50,000 DOM insertions still produces a slow page. Pagination, virtualization, incremental rendering, and a worker for heavy computation solve different bottlenecks. Debouncing limits how frequently search runs; it does not improve the complexity of one search. Memoization helps repeated inputs but can create unbounded memory growth without an eviction policy.

For a queue, repeatedly calling `shift()` can require moving entries. An array plus a head index is often simpler for a bounded traversal; occasionally compact the consumed prefix. For recursive trees, very deep input can exhaust the call stack, so an explicit stack is a useful alternative.

## Exercises and acceptance criteria

1. Build an undo stack for selection changes. Acceptance: undo restores the prior set without mutating a saved snapshot, and an empty history does nothing safely.
2. Traverse a nested menu breadth-first. Acceptance: each item appears once, a cycle does not hang the program, and sibling ordering is preserved.
3. Benchmark 100, 10,000, and 100,000 records with `performance.now()`. Acceptance: report warm-up, repeated measurements, memory tradeoffs, and separate index construction from lookup time.

## References

- [MDN: keyed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections)
- [MDN: Array sorting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)
- [MDN: Performance API](https://developer.mozilla.org/en-US/docs/Web/API/Performance)
