# 70. Compatibility and progressive enhancement

[Curriculum](../README.md)

**Prerequisites:** [HTML](../03-semantic-html/tutorial.md), [CSS](../06-css-foundations/tutorial.md), and [JavaScript](../13-javascript-fundamentals/tutorial.md).

**You will learn to:** define a support policy, detect capabilities, and preserve useful behavior when an enhancement is unavailable.

## Start with a working baseline

Progressive enhancement builds a useful core experience and adds capabilities where available. It is not merely making an attractive page look plainer in an older browser. A link should navigate, a form should have a submission model, and important content should remain understandable when an optional script fails.

A support policy states which browsers, devices, embedded webviews, assistive technologies, and connection conditions matter to the product. Use actual audience requirements and organizational constraints. “Modern browsers” is too vague to guide testing. Separate full support from a functional fallback and from an explicitly unsupported environment.

Feature detection asks whether a capability exists. User-agent detection guesses from an identity string and becomes fragile when browsers change or spoof it. Even a detected API can fail due to permissions, policy, configuration, or resource limits; detect the API and handle the operation's result.

## Tutorial: enhance visible help into a popover

Create this standalone `index.html` and serve it locally:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Progressive help</title>
<h1>Create an account</h1>
<button id="help-button" type="button" hidden>Password guidance</button>
<aside id="password-help" aria-labelledby="help-title">
  <h2 id="help-title">Password guidance</h2>
  <p>Use a long, unique password and store it in a password manager.</p>
</aside>
<style>
  body { font: 1rem/1.6 system-ui; max-width: 48rem; margin: 2rem auto; padding: 1rem; }
  aside { border: 1px solid; padding: 1rem; }
  @supports (text-wrap: balance) {
    h1, h2 { text-wrap: balance; }
  }
</style>
<script type="module">
const help = document.querySelector('#password-help');
const button = document.querySelector('#help-button');
if ('popover' in HTMLElement.prototype) {
  help.setAttribute('popover', 'auto');
  button.setAttribute('popovertarget', help.id);
  button.hidden = false;
}
</script>
</html>
```

With JavaScript disabled or the API unavailable, the guidance is visible in normal document flow. With support, the button opens the native popover and light dismissal is available. The [Popover API guide](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API) describes its behavior. A popover is not automatically a modal dialog or a menu: choose semantics and interaction according to the content.

This enhancement is deliberately small and synchronous. In a larger component, only hide the fallback after the replacement has initialized successfully. A script that hides content first and crashes halfway through setup turns an optional enhancement into a dependency.

## Handle syntax, APIs, and delivery separately

A transpiler can rewrite selected JavaScript syntax, but does not supply every missing browser API. Polyfills implement selected runtime behavior and bring download, maintenance, and fidelity costs. CSS prefixing can address some historical syntax differences but cannot recreate arbitrary layout engines. Choose build targets based on the support policy and verify the generated output.

Use `@supports` for CSS capability queries and sensible declarations before newer alternatives. Prefer logical properties and flexible layouts so content survives different fonts and languages. Avoid brittle browser-specific fixes until you have reproduced the problem and isolated the responsible behavior.

Content Security Policy, CORS, MIME types, module loading, and service workers can make a supported feature fail in production. Test the deployed headers and actual asset URLs. A browser compatibility table is useful evidence about implementation support, not proof that your particular feature works in the complete product.

## Build a test matrix

Choose a small primary matrix that reflects users and a secondary exploratory matrix for risk. Cover keyboard-only input, touch, zoom, reduced motion, high contrast or forced colors, slow connections, and missing optional capabilities. Test server-rendered pages before hydration as well as after it. Use automated cross-browser tests for stable workflows and real-device checks for hardware-sensitive behavior.

When adding a new API, record the fallback and the reason it is acceptable. If an essential capability has no acceptable fallback, communicate the product's requirement early rather than failing silently after the user has entered data.

**Exercise:** enhance a normal contact form with client-side submission while preserving its server-backed baseline. Acceptance criteria: normal submission works without JavaScript, successful enhancement prevents duplicate submissions, request failures preserve entered values, and unsupported APIs do not hide required controls.

**References:** [Feature detection](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Testing/Feature_detection), [CSS supports](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports), [Baseline](https://web.dev/baseline).
