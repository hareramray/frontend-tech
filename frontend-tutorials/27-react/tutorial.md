# 27. React: components, state, and effects

[Curriculum](../README.md)

Prerequisites: [modules](../19-modules-errors-and-debugging/tutorial.md), [build tools](../24-build-tools-and-bundlers/tutorial.md), and [component contracts](../26-components-and-design-systems/tutorial.md).

## What you will learn

Build a small interactive list with function components, props, immutable updates, stable keys, and derived values. Understand when effects are appropriate and how component identity affects state.

## Concepts and decisions

A React component describes UI for a particular set of props and state. JSX is syntax transformed into JavaScript; it is not an HTML string. Props carry inputs from a parent, while state records information a component owns between renders. A state update requests another render. Event handlers belong to the render that created them, so they see that render's snapshot.

Hooks such as `useState` are called at the top level of components or custom hooks. Lists need stable keys that identify records among siblings. Moving shared state to the nearest common parent lets multiple children use one source of truth. These basic contracts are documented in [React's Quick Start](https://react.dev/learn).

Store the minimum source state. A filtered list or remaining count can be derived during rendering. Effects synchronize with external systems such as event subscriptions or a non-React widget; they are not a default mechanism for calculating one value from another. Cleanup should undo the setup so remounting does not leave duplicate subscriptions.

## Worked tutorial: a reading checklist

Use Node.js 22.12 or newer, subject to Vite's installed engine requirement. Create a project:

```sh
npm create vite@latest react-lab -- --template react
cd react-lab
npm install
```

The command intentionally selects the current scaffolder; commit the generated lockfile for repeatable work. Keep generated `src/main.jsx` and replace `src/App.jsx` with:

```jsx
import { useState } from 'react';

function ReadingItem({ item, onToggle }) {
  return (
    <li>
      <label>
        <input type="checkbox" checked={item.read}
          onChange={() => onToggle(item.id)} />
        {item.title}
      </label>
    </li>
  );
}

export default function App() {
  const [title, setTitle] = useState('');
  const [items, setItems] = useState([
    { id: 'first', title: 'Understand render snapshots', read: false }
  ]);
  const unread = items.filter(item => !item.read).length;

  function addItem(event) {
    event.preventDefault();
    const cleaned = title.trim();
    if (!cleaned) return;
    const item = { id: crypto.randomUUID(), title: cleaned, read: false };
    setItems(previous => [...previous, item]);
    setTitle('');
  }

  function toggle(id) {
    setItems(previous => previous.map(item =>
      item.id === id ? { ...item, read: !item.read } : item
    ));
  }

  return (
    <main>
      <h1>Reading checklist</h1>
      <form onSubmit={addItem}>
        <label htmlFor="title">Reading title</label>
        <input id="title" value={title}
          onChange={event => setTitle(event.target.value)} />
        <button type="submit" disabled={!title.trim()}>Add reading</button>
      </form>
      <p role="status">{unread} unread</p>
      <ul>{items.map(item =>
        <ReadingItem key={item.id} item={item} onToggle={toggle} />
      )}</ul>
    </main>
  );
}
```

Replace `src/index.css` with simple styles so the starter presentation does not obscure the example:

```css
body { font: 1rem system-ui; margin: 2rem; }
main { max-width: 40rem; margin: auto; }
input, button { font: inherit; margin: .4rem; }
li { margin-block: .6rem; }
```

Run `npm run dev`. Add two readings and toggle one. Expect three items and two unread; the input clears after each successful submission. Run `npm run build` to validate the production compilation. The page uses local component state, so reloading resets the list.

## Why these updates work

Each list update creates a new array, and a changed item receives a new object. Unchanged objects are reused. Functional state updaters use the most recent queued state, which matters when multiple updates occur before the next render. IDs are created once when the item is created, never during list rendering. Using a position as a key can attach input state to the wrong item after insertion or sorting.

The parent owns the collection and passes behavior to each child. This keeps updates observable and makes the row reusable. If distant components need the same state, consider composition or context before adding a global store. Context distributes a value; it does not independently define ownership, caching, or update policies.

## Effects, performance, and errors

Use an effect when a component must subscribe to a browser event or synchronize with an external system. Include the reactive values it reads and return cleanup where necessary. Development Strict Mode may exercise setup and cleanup again to reveal lifecycle defects; code should tolerate that pattern. Avoid suppressing dependency warnings to force an effect to behave like a different lifecycle.

Profile before adding memoization. `useMemo` caches a calculation, and `useCallback` caches a function identity, but neither repairs incorrect state modeling. Error boundaries contain rendering failures in descendants; event-handler and asynchronous failures still require appropriate handling. Routing, data caching, and server rendering introduce additional architecture choices covered in later chapters.

## Exercises and acceptance criteria

1. Add deletion and sorting. Acceptance: checked state remains attached to the same record after either operation.
2. Add an unread-only filter. Acceptance: the filtered collection is derived, not synchronized through an effect.
3. Persist the list in local storage. Acceptance: malformed stored JSON produces a safe fallback, storage failures are handled, and the initial load does not overwrite existing data with an empty list.

## References

- [React: state as a snapshot](https://react.dev/learn/state-as-a-snapshot)
- [React: you might not need an effect](https://react.dev/learn/you-might-not-need-an-effect)
- [React: synchronizing with effects](https://react.dev/learn/synchronizing-with-effects)
