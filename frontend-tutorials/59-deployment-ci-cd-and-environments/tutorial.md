# 59. Deployment, CI/CD, and environments

[Curriculum](../README.md)

**Prerequisites:** [build tools](../24-build-tools-and-bundlers/tutorial.md), [testing](../46-unit-testing/tutorial.md), and [rendering](../41-rendering-csr-ssr-ssg-and-islands/tutorial.md).

**You will learn to:** produce a reproducible release, separate public configuration from secrets, configure hosting correctly, and recover from a failed deployment.

## Understand the release boundary

A static build produces files served by a web server or CDN. An SSR application also requires a compatible server runtime. A deployment can therefore fail even when compilation succeeds: routes may not rewrite correctly, environment variables may be missing, or a runtime adapter may not match the host. Decide the deployment target before adopting server-only framework features.

Development, preview, staging, and production are separate environments. Give them separate data and credentials. A variable substituted into browser JavaScript is public, regardless of whether its name contains `SECRET`. Keep private service credentials on the server. Public API origins, release IDs, and non-sensitive feature configuration can be exposed deliberately.

An artifact should be immutable: build it once, attach a commit identifier, validate it, and promote that artifact when the hosting model allows. Rebuilding from the same source against different dependency versions can produce different code. Commit the lockfile, select a supported runtime, and record the package-manager version.

## Tutorial: add a build gate

1. Use an existing npm application with a committed `package-lock.json`. Its `package.json` must provide `lint`, `test:ci`, and `build`. For Vitest, `test:ci` can be `vitest run`; a watch-mode test command is unsuitable for CI.
2. Put the runtime major supported by your framework in `.nvmrc`. This example uses Node 24; verify your application's engine requirements before adopting it.
3. Create `.github/workflows/check.yml`:

```yaml
name: Check frontend
on: [push, pull_request]
permissions:
  contents: read
jobs:
  verify:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version-file: .nvmrc
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm run test:ci
      - run: npm run build
```

These versioned action tags make the example readable. For stricter dependency control, resolve reviewed action versions to full commit SHAs and maintain those pins. Never give an untrusted pull request access to production secrets. GitHub explains the supported Node setup and caching workflow in its [Node.js CI guide](https://docs.github.com/en/actions/tutorials/build-and-test-code/nodejs).

4. Run the same four commands locally. Deliberately break a test on a branch and verify that the check fails. A green check only means the configured checks passed; it does not prove the application is usable.
5. Configure your chosen host to serve the generated output directory. For a client-side router, return the application HTML for valid application routes while preserving real missing-asset and API errors. For SSR, use the framework's host adapter and start command.

## Delivery details that change behavior

Use long-lived immutable caching for content-hashed assets. Keep HTML revalidatable so it discovers new asset names. Retain old assets during a rollout because an already-open page can request an older lazy-loaded chunk. Give service workers an explicit update strategy; a CDN purge does not erase a user's service-worker cache.

Enable HTTPS, compression, correct MIME types, appropriate security headers, and a canonical hostname. Validate DNS and certificate behavior on the actual domain. Configure source maps according to your debugging and disclosure requirements. Preview URLs should not accidentally index private content or connect to production data.

Rolling, blue/green, and canary deployments change how traffic reaches a release. Each needs a health check and a rollback mechanism. Database or API changes must remain compatible with both old and new clients during the transition. A frontend rollback cannot undo an incompatible server-side data migration.

## Debug and verify

Check the browser Network panel before changing application code: distinguish a missing chunk, a redirect loop, a wrong base path, and an API failure. Compare the release ID shown in telemetry with the deployed artifact. Test a direct deep link, refresh, offline recovery, a slow connection, and the previous release's open tab.

**Exercise:** deploy a small application to a preview environment and write a release checklist. Acceptance criteria: a clean checkout passes CI; a failing test blocks release; deep links work; browser bundles contain no private credential; hashed assets and HTML have appropriate different cache behavior; and you can demonstrate rollback to a known artifact.

**Further study:** container images, serverless execution limits, edge runtimes, infrastructure as code, artifact signing, deployment concurrency, dependency provenance, and cost monitoring. These belong to the same release discipline even when a hosting dashboard hides the infrastructure.

**References:** [GitHub Actions security](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions), [HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching).
