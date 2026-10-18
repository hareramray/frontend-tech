# 30. Svelte: compiled components and runes

[Curriculum](../README.md)

Prerequisites: [reactive patterns](../22-functional-and-reactive-patterns/tutorial.md), [build tools](../24-build-tools-and-bundlers/tutorial.md), and [component contracts](../26-components-and-design-systems/tutorial.md).

## What you will learn

Use Svelte 5-style runes to model state and derivations, bind form inputs, render keyed collections, and choose appropriate effect boundaries. Build an editable packing list.

## Concepts and decisions

Svelte compiles component source into JavaScript that updates the interface. A `.svelte` file contains script, markup, and style. Compilation changes how the implementation is generated; it does not remove the need to reason about state, ownership, accessibility, or lifecycle.

`$state` declares reactive state. Arrays and simple objects can become deeply reactive proxies, so supported nested mutations trigger updates. `$derived` expresses a value calculated from reactive inputs. `$props` reads component inputs, and `$effect` synchronizes side effects with reactive dependencies. Runes are compiler-recognized syntax, not ordinary functions imported from a package. See the official [`$state`](https://svelte.dev/docs/svelte/$state) and [`$derived`](https://svelte.dev/docs/svelte/$derived) references.

Keep derivations free of side effects. A filtered collection, count, or computed label usually belongs in `$derived`. Effects are appropriate for external APIs that do not participate in Svelte's normal rendering. They can return cleanup for listeners, timers, or other resources. Mixing older reactive syntax and newer rune examples without understanding their modes can create confusing expectations; this chapter consistently uses runes.

## Worked tutorial: packing list

Use Node.js 22.12 or newer, subject to the installed scaffolder's engine requirement. Run:

```sh
npm create vite@latest svelte-lab -- --template svelte
cd svelte-lab
npm install
```

Keep the generated entry point and replace `src/App.svelte`:

```svelte
<script>
  let title = $state('');
  let showUnpacked = $state(false);
  let items = $state([
    { id: 'water', title: 'Water bottle', packed: false },
    { id: 'map', title: 'Offline map', packed: true }
  ]);
  let remaining = $derived(items.filter(item => !item.packed).length);
  let visible = $derived(showUnpacked ? items.filter(item => !item.packed) : items);

  function add(event) {
    event.preventDefault();
    const cleaned = title.trim();
    if (!cleaned) return;
    items.push({ id: crypto.randomUUID(), title: cleaned, packed: false });
    title = '';
  }
</script>

<main>
  <h1>Packing list</h1>
  <form onsubmit={add}>
    <label for="title">Item to pack</label>
    <input id="title" bind:value={title}>
    <button type="submit" disabled={!title.trim()}>Add item</button>
  </form>
  <label>
    <input type="checkbox" bind:checked={showUnpacked}>
    Show unpacked only
  </label>
  <p role="status">{remaining} items left to pack</p>
  <ul>
    {#each visible as item (item.id)}
      <li>
        <label>
          <input type="checkbox" bind:checked={item.packed}>
          {item.title}
        </label>
      </li>
    {:else}
      <li>No items in this view.</li>
    {/each}
  </ul>
</main>

<style>
  main { max-width: 40rem; margin: 2rem auto; padding: 1rem; }
  input, button { font: inherit; margin: .4rem; }
  li { margin-block: .6rem; }
</style>
```

Replace generated `src/app.css` with `body { font-family: system-ui; margin: 0; }`. Run `npm run dev`, add a third item, and check Water bottle. Expect one remaining unpacked item after that sequence. Toggle the filter and observe the list update without maintaining a second mutable list. Run `npm run build` for production compilation.

The array push and nested checkbox binding are intentional: Svelte's reactive proxy tracks this state. That does not mean arbitrary external objects or every class instance automatically behave the same way. When handing data to an external library, inspect whether it expects a plain snapshot rather than a live proxy.

## Component design and application structure

Extract a packing row when it gains independent behavior. Define inputs with `$props` and callback props for actions whose ownership belongs to the parent. Two-way binding can be convenient, but expose it deliberately rather than making every child mutate its parent. Snippets and rendering composition can share markup without turning every variation into a separate component.

Svelte is the component layer; SvelteKit adds application concerns including routing, server rendering, loading, and deployment adapters. A Vite client-only lab does not exercise those server boundaries. When moving to a server-rendered application, keep browser-only APIs out of code that runs on the server and ensure initial output agrees during hydration.

## Pitfalls and debugging

Do not copy a reactive value into an ordinary variable and expect the copy to remain connected forever. Define a derivation or read the source when needed. Effects that update their own dependencies can loop; reconsider whether the value should be derived. Stable keys preserve row identity when filtering or sorting.

Compiler accessibility warnings are useful feedback, but they do not replace keyboard and assistive-technology testing. Inspect the generated DOM when an interaction fails, confirm the correct event attribute is attached, and trace which reactive read drives the visible value. Timer and listener cleanup should be tested by repeatedly mounting and removing a component.

## Exercises and acceptance criteria

1. Add deletion and alphabetical sorting. Acceptance: packed state follows item identity and counts remain correct.
2. Extract a row component. Acceptance: its ownership contract is documented and changes propagate through an explicit callback or deliberate binding.
3. Add a clock using an effect with cleanup. Acceptance: only one timer exists per mounted clock and removal stops updates.

## References

- [Svelte: component overview](https://svelte.dev/docs/svelte/overview)
- [Svelte: effects](https://svelte.dev/docs/svelte/$effect)
- [Svelte: props](https://svelte.dev/docs/svelte/$props)
- [SvelteKit: introduction](https://svelte.dev/docs/kit/introduction)
