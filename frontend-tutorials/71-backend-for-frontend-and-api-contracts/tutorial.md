# 71. Backend for frontend and API contracts

[Curriculum](../README.md)

**Prerequisites:** [HTTP](../34-http-fetch-and-rest/tutorial.md), [TypeScript](../20-typescript/tutorial.md), [authentication](../39-authentication-and-authorization/tutorial.md), and [deployment](../59-deployment-ci-cd-and-environments/tutorial.md).

**You will learn to:** distinguish browser and server responsibilities, adapt an upstream response, and define a contract that can evolve safely.

## Understand the boundary

A backend for frontend, or BFF, is a server layer shaped around a client experience. It can aggregate several services, keep service credentials out of browser bundles, normalize inconsistent data, and enforce a session-based boundary. It is useful when those responsibilities are real; adding a proxy solely to rename fields can also add latency and operational work.

The browser remains untrusted. Hiding an administrative button does not authorize an operation, and a BFF must verify identity and permissions independently. A same-origin deployment may simplify cookie and CORS behavior, but it does not remove CSRF or injection concerns. Keep private network credentials, authorization decisions, and trusted price calculations on the server.

## Tutorial: adapt an upstream catalog

This runnable Node example uses a local fixture to demonstrate the boundary without requiring an account or an external API. It exposes public demonstration data and does not implement authentication. Save it as `server.mjs` and run `node server.mjs` with a supported Node version.

```js
import { createServer } from 'node:http';

async function readUpstreamCatalog() {
  return [{ product_code: 'book-1', display_name: 'Frontend handbook', price_minor: 2500 }];
}

function toProduct(item) {
  if (typeof item !== 'object' || item === null ||
      typeof item.product_code !== 'string' ||
      typeof item.display_name !== 'string' ||
      !Number.isSafeInteger(item.price_minor) || item.price_minor < 0) {
    throw new Error('Invalid upstream product');
  }
  return { id: item.product_code, name: item.display_name, priceCents: item.price_minor };
}

const server = createServer(async (request, response) => {
  response.setHeader('Content-Type', 'application/json; charset=utf-8');
  response.setHeader('X-Content-Type-Options', 'nosniff');
  response.setHeader('Cache-Control', 'no-store');
  if (request.method !== 'GET' || request.url !== '/api/products') {
    response.writeHead(404);
    response.end(JSON.stringify({ error: { code: 'NOT_FOUND' } }));
    return;
  }
  try {
    const source = await readUpstreamCatalog();
    if (!Array.isArray(source)) throw new Error('Expected an array');
    const products = source.map(toProduct);
    response.writeHead(200);
    response.end(JSON.stringify({ products }));
  } catch (error) {
    console.error('Catalog adaptation failed', error.message);
    response.writeHead(502);
    response.end(JSON.stringify({ error: { code: 'CATALOG_UNAVAILABLE' } }));
  }
});
server.listen(3001, '127.0.0.1', () => {
  console.log('Open http://127.0.0.1:3001/api/products');
});
```

Open the printed URL. Expect one normalized product. Change `price_minor` to a string and restart; expect a 502 response with a stable error code instead of a malformed product. TypeScript types alone would not validate this external value at runtime.

To replace the fixture, fetch from a fixed, configured upstream origin, apply a timeout, check the HTTP status, parse the body, and validate its schema. Do not accept arbitrary upstream URLs from browser query parameters: that can create server-side request forgery. Forward only the headers needed by the upstream, and do not leak its credentials or raw error details to the client.

## Write and evolve the contract

A contract specifies routes, methods, parameters, request/response schemas, error codes, authorization expectations, pagination, and caching. OpenAPI describes HTTP APIs in a machine-readable format; use a specific version supported by your tooling rather than assuming every generator supports every specification revision. See the [OpenAPI specification](https://spec.openapis.org/oas/latest.html).

Generated clients improve consistency but still need runtime error handling. Validate at trust boundaries, then work with trusted internal types. Distinguish absent, null, and empty values. Define money units, date formats, time zones, and identifier stability. Document whether a retry can duplicate a mutation and provide idempotency support when necessary.

Favor additive compatible changes, but recognize that adding an enum value can break exhaustive clients. During deployments, old browser tabs can continue calling the API for hours or days. Use compatibility windows and deprecation plans. Consumer-driven contract tests can detect breaking assumptions before production, while integration tests verify the real transport and authorization boundary.

## Debug and practice

Attach a request identifier across browser, BFF, and upstream logs without recording secrets or unnecessary personal data. Separate upstream latency, adaptation time, and response transfer. Cache user-specific data only with correct partitioning; a shared cache key that omits identity can leak information.

**Exercise:** add a product-detail endpoint and an explicit response schema. Acceptance criteria: an unknown product returns a documented 404, malformed upstream data is rejected, timeouts produce a bounded failure, browser output contains no upstream credential, and an old client still understands a compatible new response.

**References:** [Node HTTP](https://nodejs.org/api/http.html), [OWASP SSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).
