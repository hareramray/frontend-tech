# 28. Vue: templates and reactive composition

[Curriculum](../README.md)

Prerequisites: [modules](../19-modules-errors-and-debugging/tutorial.md), [reactive patterns](../22-functional-and-reactive-patterns/tutorial.md), and [build tools](../24-build-tools-and-bundlers/tutorial.md).

## What you will learn

Use Vue single-file components, refs, computed values, event handlers, form bindings, and keyed lists. Build an inventory editor and distinguish derived state from side effects.

## Concepts and decisions

A Vue single-file component combines a template, script, and optional style block. The Composition API groups reactive state and behavior by feature. With `<script setup>`, top-level bindings are available to the template without manually returning an object. A `ref` wraps a reactive value: JavaScript accesses `.value`, while templates commonly unwrap refs automatically.

Computed values describe pure derivations and update from their reactive dependencies. Watchers are appropriate for effects such as synchronizing an external API or running cancellable asynchronous work. Use a computed filtered list instead of a watcher that repeatedly copies one array into another. The [Vue guide](https://vuejs.org/guide/essentials/reactivity-fundamentals.html) explains ref and reactive-object behavior.

Templates use directives for conditional rendering, repetition, binding, and event handling. `v-if` conditionally creates content, while `v-show` changes visibility through CSS. A keyed `v-for` preserves identity when records move. `v-model` coordinates input value and update events, but it does not perform business validation automatically.

## Worked tutorial: inventory quantities

Use Node.js 22.12 or newer, subject to the scaffolder's engine requirement. Run:

```sh
npm create vite@latest vue-lab -- --template vue
cd vue-lab
npm install
```

Keep the generated entry point and replace `src/App.vue`:

```vue
<script setup>
import { computed, ref } from 'vue';

const query = ref('');
const products = ref([
  { id: 'bag', name: 'Canvas bag', quantity: 2 },
  { id: 'mug', name: 'Travel mug', quantity: 0 },
  { id: 'book', name: 'Notebook', quantity: 4 }
]);

const visible = computed(() => {
  const needle = query.value.trim().toLowerCase();
  return products.value.filter(product =>
    product.name.toLowerCase().includes(needle)
  );
});
const total = computed(() => products.value.reduce(
  (sum, product) => sum + product.quantity, 0
));

function changeQuantity(id, amount) {
  const product = products.value.find(item => item.id === id);
  if (product) product.quantity = Math.max(0, product.quantity + amount);
}
</script>

<template>
  <main>
    <h1>Inventory</h1>
    <label for="search">Find a product</label>
    <input id="search" v-model="query" type="search">
    <p role="status">{{ total }} items in stock</p>
    <ul v-if="visible.length">
      <li v-for="product in visible" :key="product.id">
        <span>{{ product.name }}: {{ product.quantity }}</span>
        <button type="button" :disabled="product.quantity === 0"
          :aria-label="`Remove one ${product.name}`"
          @click="changeQuantity(product.id, -1)">−</button>
        <button type="button" :aria-label="`Add one ${product.name}`"
          @click="changeQuantity(product.id, 1)">+</button>
      </li>
    </ul>
    <p v-else>No products match your search.</p>
  </main>
</template>

<style scoped>
main { max-width: 40rem; margin: 2rem auto; padding: 1rem; }
li { display: flex; gap: .6rem; align-items: center; margin-block: .8rem; }
button, input { font: inherit; padding: .4rem; }
</style>
```

Replace generated `src/style.css` with `body { font-family: system-ui; margin: 0; }`. Run `npm run dev`, open the local URL, and increment the bag quantity. Expect the total to move from six to seven. Search for `mug`; only that row remains, while the total still represents all products. The decrement control is disabled at zero. Run `npm run build` to check production compilation.

Vue tracks the nested mutation in this example through reactive proxies. That differs from React's usual immutable state-update convention. However, shared ownership still matters: a child should not silently mutate a parent's contract. Pass props down and emit an event to request a change. Reusable reactive behavior can be extracted into a composable named with a `use` prefix.

## Scaling beyond one component

Split a product row when it has a meaningful contract or independently testable behavior. Define props and emitted events clearly. Slots allow a parent to supply content while a child owns layout and behavior. Provide/inject can distribute dependencies through a subtree; a dedicated store such as Pinia may suit shared domain state across many routes.

Template refs give access to a rendered element for focused imperative work such as moving focus after a dialog opens. Wait for the appropriate lifecycle and avoid assuming an element exists before mounting. Scoped styles limit selector matching to the component's generated scope; they do not create the same isolation boundary as native shadow DOM.

## Pitfalls and debugging

Destructuring properties from a reactive object can disconnect primitive values from their reactive source. Inspect whether you need `toRefs`, a computed value, or direct property access. Do not place asynchronous work inside a computed getter. For watchers that fetch data, cancel or ignore outdated responses and dispose subscriptions when their owner disappears.

Use Vue DevTools to inspect component state, props, and emitted events. When the UI does not update, check whether a value is reactive, whether the template reads it, and whether a local copy diverged from the source. Stable IDs remain necessary even when a list appears visually unchanged.

## Exercises and acceptance criteria

1. Extract a product row with props and emits. Acceptance: only the parent changes inventory and the child exposes accessible control names.
2. Add a low-stock filter using computed state. Acceptance: stock changes immediately affect membership without a watcher copying the list.
3. Save inventory through a mocked asynchronous function. Acceptance: loading and failure states are visible, duplicate saves are controlled, and retries preserve user edits.

## References

- [Vue: quick start](https://vuejs.org/guide/quick-start.html)
- [Vue: computed properties](https://vuejs.org/guide/essentials/computed.html)
- [Vue: composables](https://vuejs.org/guide/reusability/composables.html)
