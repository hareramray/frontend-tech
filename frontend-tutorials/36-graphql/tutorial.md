# 36. GraphQL clients and typed operations

[Curriculum](../README.md)

Prerequisites: [HTTP and Fetch](../34-http-fetch-and-rest/tutorial.md), [TypeScript boundaries](../20-typescript/tutorial.md), and [server-state caching](../35-server-state-and-caching/tutorial.md).

## What you will learn

Read a schema, write a named query with variables, distinguish transport errors from GraphQL errors, and reason about caching and pagination. Run a local GraphQL schema and inspect the exact data selected by an operation.

## Concepts and decisions

GraphQL defines a typed schema and an operation language. A query selects fields; a mutation requests a write operation; a subscription describes a stream whose transport depends on the implementation. Object fields connect types. Scalars represent leaf values. An exclamation mark marks a non-null type, while list syntax describes collections. Nullability is a contract with observable error behavior, not merely a documentation hint.

Clients select the fields they need, use arguments to parameterize fields, and pass variables separately from the operation text. Named operations improve tracing. Fragments reuse field selections, aliases distinguish repeated field requests, and directives can conditionally include fields. See the [official query guide](https://graphql.org/learn/queries/).

GraphQL does not automatically solve authorization, database efficiency, caching, or version compatibility. A single request can still trigger expensive nested work. Servers need field-level access policies, batching where appropriate, and query-cost controls. Clients should request useful data shapes without assuming every possible graph traversal is inexpensive.

## Worked tutorial: execute a catalog query locally

Use Node.js 22.12 or newer and npm. In an empty `graphql-lab` directory run:

```sh
npm init -y
npm install graphql
```

Create `catalog.mjs`:

```js
import { buildSchema, graphql } from 'graphql';

const schema = buildSchema(`
  type Product {
    id: ID!
    name: String!
    priceCents: Int!
  }
  type Query {
    product(id: ID!): Product
    products: [Product!]!
  }
`);

const products = [
  { id: 'bag', name: 'Canvas bag', priceCents: 1800 },
  { id: 'mug', name: 'Travel mug', priceCents: 1200 }
];
const rootValue = {
  product: ({ id }) => products.find(product => product.id === id) ?? null,
  products: () => products
};

const source = `
  query ProductDetails($id: ID!) {
    product(id: $id) {
      id
      name
    }
  }
`;

for (const id of ['bag', 'missing']) {
  const result = await graphql({
    schema, source, rootValue, variableValues: { id },
    operationName: 'ProductDetails'
  });
  console.log(JSON.stringify(result, null, 2));
}

const invalid = await graphql({
  schema, source: '{ products { unknownField } }', rootValue
});
console.log(invalid.errors?.map(error => error.message));
```

Run `node catalog.mjs`. The first result contains the selected product's ID and name but not its price. The second returns `product: null` because that field is nullable. The invalid operation produces a validation error before normal execution. Add `priceCents` to the query and rerun to observe the response shape change.

This lab executes the schema directly and does not start an HTTP endpoint. It isolates operation semantics from transport setup. A production frontend usually sends the same operation and variables to a server through an HTTP client or a GraphQL client library.

## Connecting a browser client

The following is an integration function, not a standalone runnable application. It requires an existing same-origin `/graphql` endpoint implementing the application's authentication and GraphQL-over-HTTP contract:

```js
export async function executeGraphQL(query, variables, signal) {
  const response = await fetch('/graphql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', Accept: 'application/graphql-response+json, application/json' },
    body: JSON.stringify({ query, variables }),
    signal
  });
  if (!response.ok) throw new Error(`HTTP ${response.status}`);
  const result = await response.json();
  if (result.errors?.length) {
    throw new Error(result.errors.map(error => error.message).join('; '));
  }
  return result.data;
}
```

This simple policy rejects any GraphQL error, including a response that also contains partial data. A real application may render usable fields and show a scoped error instead. Decide that policy explicitly. Validate unexpected response shapes at the boundary and avoid displaying internal server diagnostics directly to users.

## Caching, generation, and evolution

Document-based caching keys an operation and its variables. Normalized caching identifies entities, often by type and ID, and merges them across query results. Normalization helps shared entities stay consistent but requires correct identity and update policies. Cursor pagination also needs merge rules so pages do not duplicate or overwrite each other.

Generated TypeScript operation types can keep selected fields aligned with the schema. They improve compile-time feedback but do not authenticate a response or validate an arbitrary server at runtime. Run schema compatibility and operation checks during changes. Prefer additive evolution and deprecation before removing fields used by deployed clients.

## Pitfalls and exercises

Avoid building operations by concatenating user input; use variables. Do not assume HTTP 200 proves every requested field succeeded. A non-null violation can propagate null to an enclosing nullable boundary. A server's resolver implementation still needs authorization regardless of what the UI displays.

1. Add a named query that uses aliases to fetch two products. Acceptance: results have distinct keys and variables remain separate from operation text.
2. Make one resolver throw. Acceptance: inspect both `data` and `errors` and explain which nullable boundary contains the failure.
3. Add cursor pagination to the schema design. Acceptance: define cursor opacity, stable ordering, page metadata, and how the client merges pages without duplicates.

## References

- [GraphQL.js: getting started](https://www.graphql-js.org/docs/)
- [GraphQL: response format](https://graphql.org/learn/response/)
- [GraphQL: pagination](https://graphql.org/learn/pagination/)
