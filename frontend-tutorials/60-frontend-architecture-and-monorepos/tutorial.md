# 60. Frontend architecture and monorepos

[Curriculum](../README.md)

**Prerequisites:** [modules](../19-modules-errors-and-debugging/tutorial.md), [packages](../23-packages-and-dependency-management/tutorial.md), and [components](../26-components-and-design-systems/tutorial.md).

**You will learn to:** define dependency boundaries, build a small workspace, choose sensible abstractions, and evaluate microfrontends.

## Organize around change

Architecture controls the cost of change. A directory tree alone does not enforce architecture: imports, ownership, public interfaces, and tests do. Separate domain rules from rendering and network details so a price calculation can be tested without mounting a page. Put feature-specific code near the feature; extract shared code only when multiple consumers need the same stable behavior.

A useful application structure is `app/` for composition and routes, `features/` for user capabilities, `shared/ui/` for general presentation, and `shared/platform/` for infrastructure. This is one convention, not a universal law. The important rule is that low-level shared code must not import a high-level feature merely to reuse a constant.

A monorepo stores several packages in one repository. It enables atomic changes and shared tooling but requires boundaries to prevent a tightly coupled collection of packages. A workspace manager links packages; it is not automatically a build scheduler, release system, or access-control boundary.

## Tutorial: a two-package workspace

Create this structure in a fresh directory:

```text
package.json
apps/demo/package.json
apps/demo/main.js
packages/money/package.json
packages/money/index.js
```

Root `package.json`:

```json
{
  "name": "frontend-workspace",
  "private": true,
  "workspaces": ["apps/*", "packages/*"],
  "scripts": { "demo": "npm run start --workspace=@course/demo" }
}
```

`packages/money/package.json`:

```json
{
  "name": "@course/money",
  "version": "1.0.0",
  "type": "module",
  "exports": "./index.js"
}
```

`packages/money/index.js`:

```js
export function formatCents(cents, locale = "en-IN", currency = "INR") {
  if (!Number.isSafeInteger(cents)) throw new TypeError("Use integer cents");
  return new Intl.NumberFormat(locale, {
    style: "currency", currency
  }).format(cents / 100);
}
```

This deliberately models currencies with two fractional digits. Do not reuse its `100` divisor for every currency or assume formatting implements tax or settlement rules.

`apps/demo/package.json`:

```json
{
  "name": "@course/demo",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": { "start": "node main.js" },
  "dependencies": { "@course/money": "1.0.0" }
}
```

`apps/demo/main.js`:

```js
import { formatCents } from "@course/money";
console.log(formatCents(129900));
```

Run `npm install`, then `npm run demo` at the root. Expect a formatted amount representing 1,299 rupees. npm links matching local workspace packages during installation; its [workspace documentation](https://docs.npmjs.com/cli/v11/using-npm/workspaces/) covers running commands in selected packages. Commit the generated lockfile.

## Grow the architecture deliberately

Expose a small package API through `exports`; avoid imports into another package's internal files. Decide whether consumers use source or compiled output. TypeScript project references and a task graph can make larger builds incremental. Cache build results only when cache keys include relevant source, dependencies, tool versions, and environment inputs. A fast incorrect cache is a release defect.

Prefer an adapter at external boundaries: a `ProductRepository` interface can be implemented by a network client or a test fixture. Avoid wrapping every standard API in a new abstraction without a concrete need. Record decisions in short architecture decision records explaining context, alternatives, consequences, and reversal cost.

Microfrontends allow separately owned interfaces to be delivered independently. Approaches include route-level composition, iframes, web components, and runtime module loading. Tradeoffs include duplicated dependencies, incompatible styles, authentication coordination, version skew, navigation ownership, and accessibility across boundaries. Independently deployed code needs explicit contracts and failure containment. A modular single application is often simpler when one team owns the whole product.

## Debug and practice

Circular dependencies often indicate misplaced responsibilities; moving an interface downward can be better than adding another re-export. Inspect package resolution when local development works but publishing fails: undeclared dependencies and missing build files are common causes. Test package consumption from its public API, not just source paths.

**Exercise:** add `@course/catalog` with a pure `sortProducts` function and a demo consumer. Acceptance criteria: packages import only public entry points, the dependency graph has no cycle, each package declares its own dependencies, a clean root install runs the demo, and an architecture note explains which code remains local to the app.

**References:** [Node package entry points](https://nodejs.org/api/packages.html#package-entry-points), [TypeScript project references](https://www.typescriptlang.org/docs/handbook/project-references.html).
