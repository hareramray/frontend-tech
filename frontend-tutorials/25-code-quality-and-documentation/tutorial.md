# 25. Code quality and documentation

[Curriculum](../README.md)

Prerequisites: [packages](../23-packages-and-dependency-management/tutorial.md) and [modules](../19-modules-errors-and-debugging/tutorial.md).

## What you will learn

Give formatting, linting, type checking, testing, and documentation distinct responsibilities. Configure a small quality pipeline and write documentation that explains decisions and behavior rather than narrating obvious syntax.

## Concepts and decisions

A formatter makes layout consistent. A linter detects selected suspicious constructs and enforces conventions. A type checker proves properties within its type model. Tests exercise concrete behavior. Reviews evaluate whether the change solves the right problem and remains understandable. No single layer replaces the others, and a green pipeline does not prove the absence of bugs.

Choose rules with a useful signal-to-noise ratio. Treat an exception as a local, explained decision rather than disabling an entire category of checks. Generated files and vendored code usually need different handling from authored source. Run checks in continuous integration as well as locally so correctness does not depend on an individual's editor.

Modern ESLint uses a flat configuration file such as `eslint.config.js`; it can target file patterns and language options explicitly. See the [official configuration guide](https://eslint.org/docs/latest/use/configure/configuration-files). Avoid copying a legacy configuration without checking the installed major version and plugin compatibility.

Documentation has several audiences. A README explains setup and the first successful run. API documentation explains inputs, outputs, invariants, and errors. An architecture decision record explains why a consequential choice was made, including rejected options. A runbook explains how to diagnose and recover from operational failures. A tutorial teaches a sequence; a reference helps someone find a fact quickly.

## Worked tutorial: a checked utility package

Use Node.js 22.12 or newer and npm. In an empty `quality-lab` directory run:

```sh
npm init -y
npm pkg set type=module
npm install --save-dev eslint @eslint/js prettier
```

Create `eslint.config.js`:

```js
import js from '@eslint/js';
export default [
  { ignores: ['dist/**', 'coverage/**'] },
  js.configs.recommended,
  {
    files: ['**/*.js'],
    languageOptions: { ecmaVersion: 'latest', sourceType: 'module' },
    rules: { eqeqeq: ['error', 'always'] }
  }
];
```

Create `slug.js`:

```js
/**
 * Produce an ASCII URL segment from a display title.
 * This intentionally drops non-ASCII letters; it is not transliteration.
 * @param {string} title
 * @returns {string}
 */
export function slug(title) {
  return title.trim().toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-+|-+$/g, '');
}
```

Create `slug.test.js`:

```js
import test from 'node:test';
import assert from 'node:assert/strict';
import { slug } from './slug.js';

test('normalizes whitespace and punctuation', () => {
  assert.equal(slug('  Frontend: A Guide!  '), 'frontend-a-guide');
});
test('documents the empty output boundary', () => {
  assert.equal(slug('!!!'), '');
});
```

Run:

```sh
npx prettier --write .
npx prettier --check .
npx eslint .
node --test
```

Expect formatting verification, no lint errors, and two passing tests. Add `const unused = 1;` to the utility and rerun lint; it should fail. Remove it, change the punctuation replacement, and observe whether the behavioral test detects the regression. These experiments show why the tools have separate jobs.

Write a short README containing the supported Node version, installation command, validation commands, one import example, and the empty-output behavior. For a URL-producing application, explain where uniqueness and fallback IDs are handled: a slug generator alone cannot guarantee either. Documentation should expose that boundary before another developer accidentally relies on it.

## Maintaining quality as a project grows

Make check scripts discoverable in `package.json`, use a lockfile, and run the same commands in CI. Incremental adoption is often more effective than enabling hundreds of rules against an established codebase. Establish a baseline, fix touched areas, and prevent new violations without hiding genuine failures.

Comments are most valuable for constraints: a browser workaround, a business rule, an unusual algorithm, or an intentional tradeoff. Prefer meaningful names and short functions over comments that restate assignments. Keep documentation close to the code it describes and review both in the same change. Broken examples are defects; execute important snippets or connect them to tests when practical.

Avoid measuring quality only through coverage percentage, line count, or lint-warning count. A test that asserts an internal implementation detail can pass while user behavior breaks. Favor boundary cases, invariants, failure paths, and representative user interactions. A focused test should explain what broke when it fails.

## Exercises and acceptance criteria

1. Add a `check` script that runs all read-only checks. Acceptance: it exits nonzero for any failing stage and does not silently rewrite source.
2. Write an architecture decision record for ASCII-only slugs. Acceptance: state context, alternatives, consequences, and a condition that would justify revisiting the decision.
3. Expand the tests for repeated separators and empty titles. Acceptance: each case verifies a documented behavior, and a deliberate regression causes a clear failure.

## References

- [Prettier: installation and verification](https://prettier.io/docs/install)
- [Node.js: test runner](https://nodejs.org/api/test.html)
- [JSDoc: documentation syntax](https://jsdoc.app/about-getting-started)
