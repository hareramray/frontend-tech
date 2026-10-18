# Observability, analytics, and feature flags

[Curriculum](../README.md)

Prerequisites: [performance measurement](../50-web-performance-and-core-web-vitals/tutorial.md), [security and privacy](../40-frontend-security-and-privacy/tutorial.md), and [HTTP](../34-http-fetch-and-rest/tutorial.md).

## Learning outcomes

Design useful telemetry with minimal personal data, distinguish operational signals from product analytics, correlate frontend failures with releases, and roll out features with a dependable fallback. Observability should help explain a concrete user problem, not collect every available value.

## Core concepts

Logs describe discrete events, metrics summarize measurements, and traces connect work across boundaries. Frontend monitoring includes uncaught errors, failed requests, rendering failures, navigation timing, user-experience metrics, and domain events. Analytics answers product questions such as whether a learner completes a lesson; operational telemetry asks whether saving the lesson is failing.

Define event names, allowed properties, units, ownership, retention, and sampling before installing instrumentation. Prefer route templates such as `/projects/:id` to raw URLs containing identifiers or search queries. Never send passwords, access tokens, complete form contents, or unreviewed error objects. Connect collection behavior to the product's applicable privacy choices and requirements without assuming one universal consent rule.

## Worked tutorial: a local telemetry contract and feature flag

1. Save `telemetry-lab/index.html` and open it in a browser. The example records only to the console; it does not transmit analytics to a service.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Telemetry lab</title>
<main>
  <h1>Lesson progress</h1>
  <label><input id="variant" type="checkbox">Preview new confirmation</label>
  <button id="complete">Complete lesson</button>
  <p id="status" role="status"></p>
</main>
<script type="module">
  const buffer = [];
  const release = 'tutorial-1';
  const allowed = new Set(['lesson_completed', 'lesson_failed']);
  function record(name, properties) {
    if (!allowed.has(name)) throw new Error('Unknown telemetry event');
    const entry = {
      name,
      release,
      route: '/lesson/:slug',
      variant: properties.variant === 'new' ? 'new' : 'control',
      durationMs: Math.max(0, Math.round(properties.durationMs)),
    };
    buffer.push(entry);
    if (buffer.length > 20) buffer.shift();
    console.log('telemetry', entry);
  }

  const button = document.querySelector('#complete');
  const status = document.querySelector('#status');
  button.onclick = async () => {
    const variant = document.querySelector('#variant').checked ? 'new' : 'control';
    const start = performance.now();
    button.disabled = true;
    status.textContent = 'Saving progress…';
    try {
      // Local service simulation. Replace with a real checked request in an app.
      await new Promise(resolve => setTimeout(resolve, 150));
      status.textContent = variant === 'new' ? 'Lesson complete. Choose your next topic.' : 'Saved.';
      record('lesson_completed', { variant, durationMs: performance.now() - start });
    } catch {
      status.textContent = 'Could not save. Try again.';
      record('lesson_failed', { variant, durationMs: performance.now() - start });
    } finally {
      button.disabled = false;
    }
  };
</script>
</html>
```

2. Complete a lesson with each preview setting and inspect the console. Each event should include the variant captured when the action began, a release, route template, and measured duration. No lesson title, account identity, or arbitrary input is collected.
3. The buffer retains at most twenty records. In a production transport, add batching, size limits, backoff, and a bounded queue. Telemetry failure must not block the user's main action. `sendBeacon` can be useful for small lifecycle-related uploads, but acceptance for sending does not prove server processing or guaranteed delivery.

## Instrumentation and diagnosis

Record a stable error category, relevant operation, release identifier, and sanitized context. Upload source maps to an access-controlled monitoring service when required for diagnosis, and choose deliberately whether they are publicly served. Avoid double-reporting the same exception through several global and framework handlers.

Tracing can connect a browser action to API and backend spans. Propagate context only to intended origins, coordinate CORS, and review baggage fields for sensitive data. Browser instrumentation has environment and package-specific limitations; start with a small supported setup and verify overhead. [OpenTelemetry browser guidance](https://opentelemetry.io/docs/languages/js/getting-started/browser/)

Combine field performance data with errors and releases. A rising failed-save rate after a deployment is more actionable when segmented by browser and feature variant. Use sample counts and denominators: ten failures mean something different among twenty attempts than among a million. Set alerts around user-impacting symptoms and provide a runbook for the first investigation.

## Feature flags and experimentation

Flags separate deployment from exposure. Define a safe default when configuration is unavailable, a stable assignment strategy, an owner, and a removal date. A kill switch should be easy to exercise and must not depend on the broken feature's own initialization. Test both enabled and disabled paths, including long-lived tabs with stale configuration.

Client-visible flags are not access control. The backend must enforce entitlement, authorization, and data exposure independently. Experiments require careful assignment, exposure logging, success criteria, guardrail metrics, and interpretation; a percentage rollout alone is not a statistically valid experiment. Avoid changing a user's variant in the middle of a workflow.

## Exercises and acceptance criteria

- Add a simulated failure path and verify exactly one failure event per failed attempt, with no success event.
- Add a telemetry transport stub that rejects requests. The user action must still complete and the queue must remain bounded.
- Specify a kill switch with its fallback, owner, and expiry. Demonstrate both states in a test.
- Draft a dashboard definition for save success rate and latency. State the numerator, denominator, time window, sample size, and privacy-reviewed dimensions.

Primary references: [OpenTelemetry browser instrumentation](https://opentelemetry.io/docs/languages/js/getting-started/browser/), [MDN sendBeacon](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon), and [web-vitals](https://github.com/GoogleChrome/web-vitals). Browser instrumentation guidance reviewed on 2026-10-18.
