# 23. Packages and dependency management

[Curriculum](../README.md)

Prerequisites: [modules](../19-modules-errors-and-debugging/tutorial.md) and command-line basics.

## What you will learn

Understand package manifests, version ranges, lockfiles, scripts, peer dependencies, and reproducible installation. Create a tiny local package and consume it from another project without publishing anything.

## Concepts and decisions

`package.json` describes a project: its identity, scripts, module format, dependencies, and public entry points. Runtime dependencies support the shipped program; development dependencies support building, checking, or testing. A frontend production bundle can include code originally listed under either category, because the bundler follows imports. The distinction primarily controls installation intent rather than automatically proving what reaches the browser.

Semantic versioning uses major, minor, and patch components to communicate compatibility expectations. A range allows the package manager to choose a compatible version; it is not a guarantee that the change has no bugs. A lockfile records resolved packages and integrity metadata. Commit one lockfile for the package manager your project uses, review changes, and keep the package-manager version consistent in the team.

`npm install` can update the lockfile when resolving dependencies. `npm ci` requires an existing compatible lockfile, removes an existing `node_modules`, and fails when manifest and lockfile disagree. That makes it appropriate for clean validation environments. See [npm's ci documentation](https://docs.npmjs.com/cli/v11/commands/npm-ci/).

Peer dependencies express compatibility with a host-owned package, such as a component library's React requirement. Optional dependencies may fail installation without failing the entire install, so code must support their absence. Transitive dependencies arrive through direct dependencies and still affect security, licensing, and bundle size. Workspaces coordinate multiple local packages without publishing each intermediate edit.

## Worked tutorial: create and consume a local utility

Use Node.js 22.12 or newer and npm. Create a parent `package-lab` folder containing `money` and `app` subfolders. In `money/package.json`, write:

```json
{
  "name": "@tutorial/money",
  "version": "1.0.0",
  "type": "module",
  "exports": "./index.js",
  "files": ["index.js"],
  "license": "UNLICENSED"
}
```

Create `money/index.js`:

```js
export function formatMoney(cents, currency = 'USD') {
  if (!Number.isSafeInteger(cents)) {
    throw new TypeError('cents must be a safe integer');
  }
  return new Intl.NumberFormat('en-US', {
    style: 'currency', currency
  }).format(cents / 100);
}
```

The utility deliberately assumes a currency with two minor-unit digits. Supporting every currency requires a richer money model; this assumption belongs in the package documentation.

Create `app/package.json`:

```json
{
  "name": "package-consumer",
  "private": true,
  "type": "module",
  "scripts": { "start": "node index.js" },
  "dependencies": { "@tutorial/money": "file:../money" }
}
```

Create `app/index.js`:

```js
import { formatMoney } from '@tutorial/money';
console.log(formatMoney(1250));
```

From `app`, run `npm install` and `npm start`. Expect `$12.50`. Inspect `package-lock.json` and `npm ls`. The dependency is local, so no registry publication or account is necessary. Run `npm ci` and rerun the script to confirm the locked installation works.

From `money`, run `npm pack --dry-run`. Inspect the listed files. The `files` allowlist helps prevent accidentally including unrelated artifacts, but always inspect the actual package contents before any real publication. The consumer's `private: true` prevents accidental publication of that app through npm.

## Choosing and maintaining dependencies

Before adding a library, compare its benefit with the code, maintenance, accessibility, bundle, and upgrade costs it introduces. Check primary documentation, license, supported environments, release history, and whether a browser-native feature already covers the requirement. A large dependency is reasonable when it solves a complex, well-defined problem; a tiny package is not automatically low risk.

Inspect installation scripts and provenance when evaluating unfamiliar packages. Installation may execute lifecycle scripts; a lockfile verifies downloaded content consistency, not the trustworthiness of that content. Avoid putting secrets in project scripts or committed configuration. Registry authentication belongs in the environment or a suitable credential mechanism.

Upgrade deliberately: identify breaking changes, change a coherent group of packages, install, inspect the lockfile, run validation, and exercise affected flows. Overrides can temporarily force a transitive version, but must remain compatible with consumers and should have a documented removal condition. Audit reports need investigation: assess reachability and impact instead of blindly applying every forced upgrade.

## Pitfalls and exercises

If a package resolves in an editor but fails in Node, inspect `exports`, module format, extension rules, and the working directory. A deep import into an unexported internal file is not a stable public contract. If multiple framework copies appear, inspect the dependency tree and peer requirements.

1. Add a second exported utility. Acceptance: consumers use only declared entry points and existing imports still work.
2. Intentionally change the manifest without regenerating the lockfile. Acceptance: document the clean-install failure and repair it using a normal install.
3. Compare two packages for one need. Acceptance: record functionality, license, maintenance, dependency count, measured bundle impact, and a reasoned choice.

## References

- [npm: package.json fields](https://docs.npmjs.com/cli/v11/configuring-npm/package-json)
- [npm: workspaces](https://docs.npmjs.com/cli/v11/using-npm/workspaces)
- [Node.js: package entry points](https://nodejs.org/api/packages.html)
