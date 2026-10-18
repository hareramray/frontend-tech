# 20. TypeScript and trustworthy boundaries

[Curriculum](../README.md)

Prerequisites: [modules and errors](../19-modules-errors-and-debugging/tutorial.md), JavaScript objects and functions.

## What you will learn

Use inference, unions, generics, and narrowing to make invalid states harder to represent. Separate compile-time checking from runtime validation, and compile a small command-line program without a framework.

## Concepts and decisions

TypeScript checks JavaScript before execution; its types do not validate a network response at runtime. A declaration such as `response as User` changes the compiler's belief without changing the value. Treat external data as `unknown` until validation establishes its shape. Prefer `unknown` over `any` at uncertain boundaries because it forces you to prove what operations are safe.

Structural typing compares shapes rather than requiring class inheritance. Interfaces describe object contracts and can merge; type aliases also express unions, tuples, and mapped types. Use whichever communicates the intended contract. Literal unions such as `'idle' | 'loading'` are useful for finite states. Discriminated unions let a property identify which additional fields exist. Narrowing follows conditions such as `typeof`, property checks, and discriminant comparisons; see the [official narrowing guide](https://www.typescriptlang.org/docs/handbook/2/narrowing.html).

Optional properties and `undefined` require deliberate handling. A non-null assertion postpones a possible crash; a guard handles it. Generics express a relationship between types, such as preserving an array's element type, rather than merely accepting arbitrary input. Avoid elaborate type puzzles when a readable concrete interface would suffice.

## Worked tutorial: validate a task payload

Use Node.js 22.12 or newer and npm. In an empty `typescript-lab` directory run:

```sh
npm init -y
npm install --save-dev typescript
```

Commit the generated lockfile when keeping the project. Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "outDir": "dist"
  },
  "include": ["src/**/*.ts"]
}
```

Create the `src` directory and `src/main.ts`:

```ts
type Task = { id: number; title: string; done: boolean };
type Result<T> =
  | { ok: true; value: T }
  | { ok: false; error: string };

function parseTask(input: unknown): Result<Task> {
  if (typeof input !== 'object' || input === null) {
    return { ok: false, error: 'Expected an object' };
  }
  if (!('id' in input) || typeof input.id !== 'number' ||
      !Number.isSafeInteger(input.id) || input.id < 1 ||
      !('title' in input) || typeof input.title !== 'string' ||
      !input.title.trim() ||
      !('done' in input) || typeof input.done !== 'boolean') {
    return { ok: false, error: 'Invalid task fields' };
  }
  return {
    ok: true,
    value: { id: input.id, title: input.title.trim(), done: input.done }
  };
}

const payload: unknown = JSON.parse('{"id":1,"title":"Learn types","done":false}');
const result = parseTask(payload);
if (result.ok) {
  console.log(result.value.title);
} else {
  console.error(result.error);
}
```

Run `npx tsc --noEmit` to check, then `npx tsc` and `node dist/main.js` to execute. Expect `Learn types`. Change the JSON ID to a string: compilation still succeeds because input data is unknown, and execution prints `Invalid task fields`. Move `result.value` outside the successful branch and run the checker again; it should report that not every result contains a value. These two experiments demonstrate different failure layers.

## Applying types in frontend code

Model component props around supported combinations. A loading result should not require invented placeholder data. Use readonly arrays to communicate that a consumer should not mutate its input. `Pick`, `Omit`, `Partial`, `Record`, and indexed access types can derive contracts, but a server update payload often deserves its own explicit type. Making every field optional can accidentally permit meaningless requests.

For DOM elements, narrow `document.querySelector()` results before using specialized properties. For events, distinguish `target`, which originated the event, from `currentTarget`, whose listener is running. Framework event types differ, so inspect the framework's definitions rather than copying another framework's annotations.

Keep `tsc --noEmit` in validation even when a bundler strips types successfully. Build transformation and semantic checking are separate operations. Generate declaration files when publishing a library; applications normally need only internal checking. Path aliases also need runtime or bundler support: compiler configuration alone cannot make a browser resolve an invented import path.

## Pitfalls, debugging, and exercises

An assertion may be justified after a separately verified invariant, but repeated assertions often signal a weak data model. Avoid treating `never` errors as nuisances: they can expose an unhandled union member. When a generic error becomes unreadable, reduce the expression into named intermediate values and inspect inferred types.

1. Add a `deleted` result variant with a timestamp. Acceptance: every switch is exhaustive, and removing a branch produces a type error through a `never` check.
2. Add optional `dueDate`. Acceptance: absent is valid, malformed strings are rejected at runtime, and `undefined` is not silently treated as a supplied date.
3. Write `first<T>(items: readonly T[]): T | undefined`. Acceptance: empty input is handled by the caller and an input of numbers never returns an unchecked string.

## References

- [TypeScript: everyday types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript: generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript: compiler options](https://www.typescriptlang.org/tsconfig/)
