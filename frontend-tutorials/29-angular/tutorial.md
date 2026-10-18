# 29. Angular: standalone components and signals

[Curriculum](../README.md)

Prerequisites: [TypeScript](../20-typescript/tutorial.md), [reactive patterns](../22-functional-and-reactive-patterns/tutorial.md), and [components](../26-components-and-design-systems/tutorial.md).

## What you will learn

Bootstrap a standalone Angular component, use signals and computed values, render keyed collections, and understand dependency injection and service boundaries. Build a small task board with explicit ownership.

## Concepts and decisions

Angular combines components, templates, dependency injection, routing, forms, and build tooling within a coordinated framework. A component class supplies state and behavior; its metadata connects the template and styles. Standalone components declare their own imports instead of requiring an application-level NgModule for every feature.

A signal stores a value that consumers read by calling it. `.set()` replaces the value and `.update()` calculates a new value from the old one. `computed()` describes a derived value. Angular tracks signal reads in reactive contexts, including templates. Computed values should describe state relationships; effects are for synchronization with external systems. See the [Angular signals guide](https://angular.dev/guide/signals).

Dependency injection gives a component access to services without constructing every dependency itself. Provider scope defines lifetime and sharing: an application-wide service and a component-provided service have different ownership. Keep presentation choices in components and reusable domain or data-access behavior in appropriately scoped services.

## Worked tutorial: a standalone task board

Install a Node.js release supported by the Angular CLI version you use; check [Angular's compatibility table](https://angular.dev/reference/versions) before setup. Use npm and run:

```sh
npm exec --package=@angular/cli -- ng new angular-lab --standalone --routing=false --style=css --skip-git --defaults
cd angular-lab
```

The CLI creates the workspace and installs dependencies. Preserve its lockfile. Create `src/task-board.ts`:

```ts
import { Component, computed, signal } from '@angular/core';

type Task = { id: string; title: string; done: boolean };

@Component({
  selector: 'app-root',
  standalone: true,
  template: `
    <main>
      <h1>Task board</h1>
      <form (submit)="add($event, title)">
        <label for="title">Task title</label>
        <input id="title" #title required>
        <button type="submit">Add task</button>
      </form>
      <p role="status">{{ remaining() }} tasks remaining</p>
      <ul>
        @for (task of tasks(); track task.id) {
          <li>
            <label>
              <input type="checkbox" [checked]="task.done"
                (change)="toggle(task.id)">
              {{ task.title }}
            </label>
          </li>
        } @empty {
          <li>No tasks yet.</li>
        }
      </ul>
    </main>
  `,
  styles: [`
    main { max-width: 40rem; margin: 2rem auto; font-family: system-ui; }
    input, button { font: inherit; margin: .4rem; }
    li { margin-block: .6rem; }
  `]
})
export class TaskBoard {
  readonly tasks = signal<Task[]>([]);
  readonly remaining = computed(() =>
    this.tasks().filter(task => !task.done).length
  );

  add(event: Event, input: HTMLInputElement): void {
    event.preventDefault();
    const title = input.value.trim();
    if (!title) return;
    const task = { id: crypto.randomUUID(), title, done: false };
    this.tasks.update(tasks => [...tasks, task]);
    input.value = '';
    input.focus();
  }

  toggle(id: string): void {
    this.tasks.update(tasks => tasks.map(task =>
      task.id === id ? { ...task, done: !task.done } : task
    ));
  }
}
```

Replace `src/main.ts` with:

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { TaskBoard } from './task-board';

bootstrapApplication(TaskBoard).catch(error => console.error(error));
```

This entry point intentionally uses only the new standalone component; generated demonstration components can remain unused. Run `npm start` and open the URL printed by the CLI. Add two tasks and check one. Expect one remaining task, preserved record identity, and focus returned to the title input after submission. Run `npm run build` to verify compilation and template checking.

## Understanding the template

Interpolation displays values. Bracket binding sets a DOM property such as `checked`. Parentheses attach an event handler. The template reference `#title` supplies the input element to the handler. `@for` tracks records through `task.id`; the `@empty` block describes the zero-record state. Each construct expresses a different responsibility, so avoid combining side effects into template expressions.

The task count is computed from the task list. Mutating the existing array without a signal update could make notification behavior unclear; replacing the changed branches keeps the update explicit. For more elaborate forms, Angular's form APIs provide validation and control state. For asynchronous streams, RxJS remains useful, especially when cancellation and event composition are central.

## Architecture, pitfalls, and debugging

Do not turn every component into a service consumer without considering its inputs and outputs. A presentational component can remain easy to reuse when a parent supplies data and handles events. Keep injection scope deliberate so unrelated routes do not accidentally share transient state.

Inspect template diagnostics before adding non-null assertions. Missing imports, mismatched event types, and invalid bindings should be corrected at their source. Use Angular DevTools to inspect components and reactive dependencies. When asynchronous work survives navigation, inspect cleanup ownership; subscriptions and timers should end with their owner or be intentionally application-scoped.

## Exercises and acceptance criteria

1. Extract task storage into a service. Acceptance: updates go through explicit methods and consumers cannot directly replace the writable signal.
2. Add a priority filter with a computed value. Acceptance: the original collection remains intact and the remaining count has a documented scope.
3. Add an asynchronous save. Acceptance: pending, success, and failure are rendered, and navigating away cannot cause an unowned update.

## References

- [Angular: local setup](https://angular.dev/tools/cli/setup-local)
- [Angular: template control flow](https://angular.dev/guide/templates/control-flow)
- [Angular: dependency injection](https://angular.dev/guide/di)
