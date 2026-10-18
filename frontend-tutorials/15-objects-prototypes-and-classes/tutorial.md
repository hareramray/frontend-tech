# Objects, prototypes, and classes

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md) and [functions and closures](../14-functions-scope-and-closures/tutorial.md). You will model an object with invariants, distinguish own and inherited properties, understand class syntax over prototypes, and choose composition when inheritance adds unnecessary coupling.

## Objects and property lookup

An object groups properties whose keys are strings or symbols. Dot notation is convenient for known identifier-like keys; bracket notation evaluates a key dynamically. Two separately created objects are unequal by identity even if their fields match. Assigning an object to another variable shares the reference rather than copying its contents.

An object's prototype supplies inherited properties. When a property is missing on the object itself, lookup can continue through the prototype chain until it reaches null. `Object.hasOwn(object, key)` checks ownership; `key in object` also considers inherited properties. Enumerability controls whether properties appear in operations such as `Object.keys()` and object spread. Getters and setters are accessor properties and may execute code when a property is read or assigned.

Class syntax creates constructors and prototype methods with clear rules. Instance fields are normally own properties, methods are normally on the prototype, and static members belong to the class constructor. `new` constructs an instance and invokes initialization. Private fields use `#` names and are enforced by the language rather than merely hidden by a naming convention. Classes are not hoisted for use before initialization and their bodies run in strict mode.

Inheritance models an “is-a” relationship only when the subtype can honor the parent contract. Extending a class just to reuse a helper often creates unnecessary coupling. Composition gives an object collaborators through fields or parameters and is frequently easier to vary. Plain records are also appropriate: data transferred through JSON should not require a class hierarchy to be understood.

## Worked tutorial: inventory with a protected invariant

1. Save this code as `app.js` and run `node app.js`, or use the browser wrapper in [JavaScript fundamentals](../13-javascript-fundamentals/tutorial.md).

```javascript
'use strict';

class InventoryItem {
  #stock;

  constructor(label, initialStock) {
    if (typeof label !== 'string' || label.trim() === '') {
      throw new TypeError('A nonempty label is required.');
    }
    if (!Number.isSafeInteger(initialStock) || initialStock < 0) {
      throw new RangeError('Stock must be a nonnegative safe integer.');
    }
    this.label = label.trim();
    this.#stock = initialStock;
  }

  get stock() {
    return this.#stock;
  }

  reserve(quantity = 1) {
    if (!Number.isSafeInteger(quantity) || quantity < 1) {
      throw new RangeError('Reservation quantity must be a positive safe integer.');
    }
    if (quantity > this.#stock) {
      throw new RangeError('Not enough stock.');
    }
    this.#stock -= quantity;
    return this.snapshot();
  }

  snapshot() {
    return Object.freeze({ label: this.label, stock: this.#stock });
  }
}

const notebook = new InventoryItem('Notebook', 5);
const pencil = new InventoryItem('Pencil', 10);
console.log(notebook.reserve(2));
console.assert(notebook.stock === 3);
console.assert(pencil.stock === 10);
console.assert(Object.hasOwn(notebook, 'label'));
console.assert(!Object.hasOwn(notebook, 'reserve'));
console.assert(Object.getPrototypeOf(notebook) === InventoryItem.prototype);
console.assert(notebook.reserve === pencil.reserve);

try {
  notebook.reserve(99);
} catch (error) {
  console.log(error.message);
}
console.assert(notebook.stock === 3, 'Rejected operations must not change stock.');

const reserveNotebook = notebook.reserve.bind(notebook);
console.log(reserveNotebook(1));
console.log(JSON.stringify(notebook.snapshot()));
```

2. Expect the first snapshot to contain stock `3`, followed by `Not enough stock.`, then a stock `2` snapshot and its JSON representation.
3. Inspect `notebook` in developer tools. `label` is public instance data; `reserve` is shared through the prototype; `stock` is exposed by an accessor backed by a private field.
4. Try passing `notebook.reserve` as a detached callback in a separate experiment. Explain why binding the receiver is required for private-field access.

Validation occurs before mutation, preserving the invariant that stock never becomes negative. The snapshot is a new plain object, so callers cannot modify the internal stock by editing a returned record. `Object.freeze()` is shallow, which is sufficient for this snapshot's primitive fields but would not freeze nested objects automatically. The public label remains writable; use a getter/private field too if its invariant must persist after construction.

## Advanced concerns and debugging

Property descriptors control writability, enumerability, configurability, or accessors. `Object.defineProperty()` is useful for deliberate APIs but often excessive for ordinary application records. `Object.create(null)` creates a dictionary without inherited object properties, while `Map` handles arbitrary keys more directly. Never blindly merge untrusted keys into sensitive object structures; keys such as `__proto__` can interact with unsafe assignment patterns and create prototype-pollution vulnerabilities.

JSON serialization omits methods and does not preserve prototypes, undefined fields, symbols, Maps, or many other JavaScript types as their original forms. `structuredClone()` supports more data types but is not a universal clone for functions, DOM nodes, or arbitrary class behavior. Define explicit transport records rather than expecting serialization to preserve an object model.

`instanceof` tests a prototype relationship, not a universal description of data shape, and can be surprising across realms such as iframes. Prefer behavioral or schema validation at external boundaries. Proxies intercept operations and can support reactive systems, but they add complexity around identity, invariants, accessors, and private fields. Learn the basic object model before introducing them.

## Exercises and acceptance criteria

- Add a `restock(quantity)` method that rejects invalid or unsafe totals. Failed operations must leave stock unchanged.
- Make `label` immutable through a private field and getter. External assignment must not alter it.
- Create a plain transport record and reconstruct a validated instance from it. Invalid persisted stock must be rejected.
- Compare inheritance and composition for adding audit logging. Choose an approach and explain how it preserves the reservation contract.

## References

- [MDN: working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN: prototype chain](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain)
- [MDN: classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)
- [MDN: private elements](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/Private_elements)
