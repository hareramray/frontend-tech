# Authentication and authorization

[Curriculum](../README.md)

Prerequisites: [JavaScript promises](../18-promises-async-and-the-event-loop/tutorial.md), [HTTP requests](../34-http-fetch-and-rest/tutorial.md), cookies, and the [security chapter](../40-frontend-security-and-privacy/tutorial.md). Study the security chapter alongside this one.

## Learning outcomes

Distinguish identity from permission, model an authentication interface without information leaks, understand session and token tradeoffs, and explain which checks belong on the server. The worked example implements the browser's state transitions; it deliberately does not implement an identity provider.

## Concepts and boundaries

Authentication establishes who is making a request. Authorization decides whether that identity can perform a particular operation on a particular resource. A signed-in editor may edit their team's article but must not automatically edit every article. Hiding an Edit button improves the interface; the endpoint must independently check the permission and resource ownership on every request.

A session cookie can carry an opaque identifier whose meaning is held on the server. In an HTTPS deployment, appropriate `Secure`, `HttpOnly`, and `SameSite` attributes reduce different risks. `HttpOnly` prevents JavaScript from reading the cookie, but injected JavaScript could still make authenticated requests. Rotate sessions after authentication and relevant privilege changes, expire them server-side, and invalidate them on logout. Do not put session identifiers in URLs or logs. [OWASP session management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

OAuth delegates access; OpenID Connect adds an identity layer. Use a maintained integration for authorization-code flow with PKCE, exact redirect registration, response validation, and transaction binding. A browser application cannot keep a client secret. Avoid treating a decoded JWT as verified identity: the receiving server must validate the signature, issuer, audience, expiry, and application policy. Refresh-token handling needs an explicit theft and replay strategy. [OAuth security best practice](https://www.rfc-editor.org/rfc/rfc9700.html)

## Worked tutorial: a session-aware interface

1. Create an empty `auth-lab` directory and save the following complete file as `index.html`. Open it in a modern browser. No dependencies or build commands are required.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Session interface lab</title>
<main>
  <h1>Account</h1>
  <p id="status" role="status"></p>
  <button id="login">Simulate sign in</button>
  <button id="logout" hidden>Simulate sign out</button>
  <button id="edit" hidden>Edit article</button>
</main>
<script type="module">
  const status = document.querySelector('#status');
  const login = document.querySelector('#login');
  const logout = document.querySelector('#logout');
  const edit = document.querySelector('#edit');
  let revision = 0;
  // Teaching fixture only: this is not authentication or access control.
  let fixtureUser = null;
  const mockSession = async () => {
    await new Promise(resolve => setTimeout(resolve, 300));
    return fixtureUser;
  };

  async function refresh() {
    const request = ++revision;
    status.textContent = 'Checking your session…';
    login.disabled = logout.disabled = true;
    edit.hidden = true;
    try {
      const user = await mockSession();
      if (request !== revision) return;
      status.textContent = user ? `Signed in as ${user.name}` : 'Signed out';
      login.hidden = Boolean(user);
      logout.hidden = !user;
      edit.hidden = !user?.permissions.includes('article:edit');
    } catch {
      if (request !== revision) return;
      status.textContent = 'Session unavailable. Try again.';
      login.hidden = logout.hidden = false;
    } finally {
      if (request === revision) login.disabled = logout.disabled = false;
    }
  }
  login.onclick = () => {
    fixtureUser = { name: 'Asha', permissions: ['article:edit'] };
    refresh();
  };
  logout.onclick = () => { fixtureUser = null; refresh(); };
  edit.onclick = () => { status.textContent = 'Demo editor opened'; };
  refresh();
</script>
</html>
```

2. Sign in and sign out. The privileged control disappears while a session check is pending and after logout. Change the fixture permissions to an empty array: the account is authenticated but cannot edit.
3. Replace `mockSession` only when a backend exists. Its contract should return a minimal account object, represent expired sessions consistently, and prevent shared caching of personal responses. Use same-origin credentialed requests where possible. A network failure is different from a confirmed signed-out response.

For cookie-authenticated writes, use the backend framework's CSRF protection and validate requests server-side. SameSite is a useful additional layer, not a universal replacement for CSRF defenses. Logout should be a protected state-changing request. [OWASP CSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)

## Advanced practice, pitfalls, and debugging

Model unknown, checking, anonymous, authenticated, refreshing, and failed states explicitly. Otherwise a page can briefly display private controls before the session resolves. Abort obsolete network requests and ignore stale responses when accounts change. Clear account-specific query caches after logout and tenant switching; a correct server does not prevent stale private data remaining onscreen.

For a backend-for-frontend architecture, the server handles provider tokens and the browser receives a session cookie. This reduces token exposure but introduces server operations and cookie/CSRF concerns. A direct browser-to-API design needs carefully scoped tokens and a storage threat model; moving tokens from local storage to memory does not eliminate XSS.

Inspect request status, cookie attributes, request origin, and redirect paths without copying credentials into tickets. Differentiate 401, 403, offline errors, and cancelled requests. Review account recovery, multi-factor prompts, passkeys, session revocation, tenant isolation, and step-up authentication as separate flows.

## Exercises and acceptance criteria

- Add a Retry control for a simulated session failure. Passing behavior retains no privileged controls and recovers without reloading.
- Introduce two delayed responses in reverse order. Passing behavior never restores an older account after a newer logout.
- Write a backend test plan: an anonymous request, another user's resource ID, and an expired session must each be rejected independently of the UI.

The primary references linked above were reviewed on 2026-10-18. Integrate a maintained provider rather than promoting this local fixture into a production login system.
