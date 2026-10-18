# Project 04. Ship and maintain a production-style application

[Projects](../README.md) · [Curriculum](../../README.md)

**Prerequisites:** complete one earlier project, one framework chapter, and the relevant security, testing, rendering, deployment, and API-contract chapters.

**Outcome:** a support-ticket application with a public information area, authenticated ticket workflows, a documented API, automated checks, observability, and a demonstrated rollback. Use synthetic users and data during the exercise.

## Milestone 1: write a small product brief

Define three journeys: a visitor reads support guidance, a signed-in user creates and tracks a ticket, and an authorized staff user updates its status. Specify which data each user may read or modify. Write acceptance criteria for loading, empty, error, expired-session, and denied-access states.

Keep the first release narrow: ticket title, description, status, author, timestamps, and a simple comment thread. Defer attachments, full-text search, collaborative editing, and complex permissions until the basic boundaries work. This is scope control, not a reason to skip quality requirements.

Choose one stack: React/Next.js, Vue/Nuxt, Svelte/SvelteKit, Angular with an appropriate server, or another explicitly justified combination. Choose rendering per route. Public guidance may be static; personalized tickets require correct identity-aware data handling. Do not put authenticated responses into a shared public cache.

## Milestone 2: define the contract and threat boundaries

Document endpoints or server actions for list, detail, create, comment, and status change. Specify stable IDs, validation limits, pagination, error codes, authorization, and idempotency behavior. For example:

```text
GET /api/tickets?cursor=... -> tickets visible to the current user
GET /api/tickets/{id}      -> one authorized ticket or a documented failure
POST /api/tickets         -> create after server validation
POST /api/tickets/{id}/comments -> add an authorized comment
PATCH /api/tickets/{id}    -> authorized status transition
```

This is a contract sketch, not a running server. Implement it using the chosen backend and verify every route independently of the UI. A user who edits a request ID must not gain access to another user's ticket. A disabled button is not a server policy.

Use the authentication provider's supported flow or a well-understood session implementation. Keep service credentials server-side. Define CSRF protection, output escaping, rate limiting, and safe logging. Avoid storing raw passwords in tutorial fixtures or building a custom cryptographic protocol.

## Milestone 3: build a complete vertical slice

Implement create-ticket from form through server validation to persisted data and back to an accessible confirmation. Use a schema or explicit validation at the trust boundary. Associate field errors with controls, preserve entered values on failure, and prevent duplicate submissions while a request is pending.

Then add list and detail routes. Store non-sensitive filters in the URL. Keep server data and temporary UI state separate. Handle expired sessions without exposing a previous user's cached data. Test account switching and logout after data has been loaded.

**Check:** a fresh browser session can complete the intended journey; a direct detail URL works; an unauthorized API call fails; and refresh does not lose a successfully created ticket.

## Milestone 4: test meaningful behavior

Use unit tests for status-transition rules and pure formatting. Use integration tests for server validation and permission checks. Use component tests for form errors and loading behavior. Use an end-to-end test for creating a ticket and then finding it in the list. Keep tests isolated with deterministic synthetic fixtures.

Add a regression test for a deliberately introduced race or duplicate-submission defect. Inspect a failing browser trace rather than increasing every timeout. Test keyboard operation and a short screen-reader journey; automated accessibility checks do not replace those passes.

Measure a representative production build on a realistic device/network profile. Record initial JavaScript size, image policy, visible loading behavior, and the relevant performance measurements. Use field metrics if the application later receives real traffic; do not treat one local score as a universal result.

## Milestone 5: release with evidence

Create a CI workflow that installs from a lockfile, checks types and code quality, runs meaningful tests, and builds. Deploy a preview first. Verify environment configuration, deep links, cookies, HTTPS, security headers, and cache behavior on the actual host. Add a release identifier to error reports and logs without recording ticket contents unnecessarily.

Define a small operational dashboard: request failures, server latency, client errors, and completion of the primary journey. Set alerts based on user impact and expected traffic, not every isolated warning. Keep analytics collection proportionate and transparent.

Prepare a rollback to the prior artifact. Introduce a harmless failing preview release and demonstrate recovery. If a database migration is involved, document compatibility with old clients and whether rollback needs a forward repair. Rolling back JavaScript cannot reverse every data change.

## Milestone 6: review and maintain

Write a short architecture decision record, API contract, local setup guide, release checklist, and incident note from the rollback exercise. Document unresolved limitations with concrete impact. Schedule dependency maintenance and removal of temporary feature flags.

**Completion criteria:** all three journeys work; authorization is enforced server-side; failures preserve user work; accessible interaction is verified; checks pass from a clean checkout; the deployed build is identifiable; private data stays out of shared caches and logs; and rollback has been demonstrated.

**Stretch work:** add attachment processing, audited staff actions, internationalization, offline drafts, or a second client. Add one capability at a time and revisit the contract, security boundary, performance budget, and test strategy for each.
