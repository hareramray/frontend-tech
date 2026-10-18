# Validation record

[Curriculum](README.md)

Prepared on **18 October 2026**. This record distinguishes documentation checks, executable-example checks, and work that still depends on a learner's browser or deployment environment.

## Library structure

- 72 numbered topic folders, each containing `tutorial.md`.
- Four project folders, each containing a guided `tutorial.md`.
- Repository README, curriculum index, setup guide, roadmap, coverage map, cheatsheets, glossary, troubleshooting guide, project index, and this validation record.
- All local Markdown file links resolve. Numbered curriculum entries map to existing tutorials.
- Markdown code fences are balanced. Every tutorial has at least 450 words of explanatory prose; the full library has more than 59,000 prose words, excluding fenced code.

## Static code checks

The checks used Node **24.19.0**. All 51 fenced JavaScript examples and 22 inline HTML script blocks passed JavaScript syntax checking. Eight JSON or JSON-LD blocks parsed successfully. These checks verify syntax, not browser behavior or complete application integration.

Selected framework examples were also checked with their relevant tooling: TypeScript type checking, React and Query JSX compilation, Vue single-file-component script/template compilation, Svelte compilation, and Angular strict template compilation. These selected compilation checks do not imply that every framework chapter was scaffolded, deployed, or tested end to end.

## Executed examples

| Area | Checks performed | Result |
|---|---|---|
| JavaScript foundations, chapters 13–16 | Ran complete examples and their assertions | Passed |
| Modules, algorithms, functional patterns, state, GraphQL: chapters 19, 21, 22, 33, 36 | Executed the self-contained Node labs | Passed |
| Package management, chapter 23 | Install, clean install, and package import | Passed |
| Code quality, chapter 25 | Lint and two example tests | Passed |
| Routing, chapter 32 | Simulated missing routes and inherited property names such as `constructor` and `__proto__` | Correct not-found behavior |
| SSE, chapter 37 | Local server smoke test and simulated hide/restore/stop/stale-event lifecycle | Passed |
| Unit and component tests, chapters 46–47 | Extracted examples into a temporary npm project and ran Vitest with the stated DOM testing dependencies | Two files and 13 tests passed |
| BFF, chapter 71 | Started the extracted server on an ephemeral loopback port; requested valid, missing, and malformed-upstream paths | Expected 200, 404, and 502 behavior |
| Maintenance, chapter 72 | Ran the regression test against both the intentionally broken and corrected implementation | Failed before the fix and passed afterward |

The chapter 46–47 run used Vitest 5.0.5 with jsdom, Testing Library DOM, and user-event. Temporary validation dependencies are not part of this Markdown library. A DOM simulation is not a full browser or an accessibility audit.

## Review corrections

Review corrected an IndexedDB open-error path, stale worker error handling, inherited-property route lookup, SSE reconnection after browser back/forward cache restoration, and pinch-zoom support on the swipe example. The coverage and index labels were reviewed against the tutorial contents.

## Limits

No complete automated real-browser pass was performed across all examples. Camera, microphone, graphics hardware, WebRTC connectivity, touch behavior, screen readers, service-worker lifecycle, cloud hosting, authentication providers, and cross-device behavior need the manual or integration checks described in their chapters.

The four projects are guided implementations, not finished deployed applications. External reference links point to primary documentation, but every remote URL was not exhaustively tested for permanent availability. Framework defaults and browser compatibility can change; use the installed versions and linked official documentation when applying an example.

Passing these checks provides useful evidence about the library and selected examples. It does not establish production readiness for an application built from the material without its own validation.
