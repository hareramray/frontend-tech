# 37. Realtime updates: WebSockets and server-sent events

[Curriculum](../README.md)

Prerequisites: [HTTP and Fetch](../34-http-fetch-and-rest/tutorial.md), asynchronous JavaScript, and [state ownership](../33-client-state-management/tutorial.md).

## What you will learn

Choose between polling, server-sent events, and WebSockets; manage connection lifetime; and plan reconnection, ordering, and missed-message recovery. Build a live event stream with a local Node server.

## Concepts and decisions

Polling asks for updates at intervals. It is simple and often sufficient when updates are infrequent. Long polling holds a request until data is available. Server-sent events, or SSE, stream text events from server to browser over HTTP. WebSockets support messages in both directions over a persistent connection. Choose according to direction, latency, infrastructure, data volume, and operational complexity.

The browser's `EventSource` reads an SSE endpoint and normally reconnects after interruption. SSE frames use fields such as `event`, `data`, `id`, and `retry`, separated by blank lines. Named events require matching event listeners. See [MDN's SSE guide](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events).

Reconnection is not equivalent to recovery. A client needs a way to identify the last processed event and recover missed changes. The server must retain or reconstruct relevant history for resume to work. Otherwise reconnect by fetching a fresh snapshot and then reattaching the stream with a defined consistency strategy.

## Worked tutorial: a live clock stream

Use Node.js 22.12 or newer and a browser. Create `server.mjs` and `index.html` in an empty directory. No dependencies are required.

```js
// server.mjs
import { createServer } from 'node:http';
import { readFile } from 'node:fs/promises';

const page = await readFile(new URL('./index.html', import.meta.url));
createServer((request, response) => {
  if (request.url === '/events') {
    response.writeHead(200, {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      Connection: 'keep-alive'
    });
    response.write('retry: 2000\n\n');
    let sequence = 0;
    const timer = setInterval(() => {
      const payload = JSON.stringify({ time: new Date().toISOString() });
      response.write(`id: ${++sequence}\nevent: tick\ndata: ${payload}\n\n`);
    }, 1000);
    response.on('close', () => clearInterval(timer));
  } else if (request.url === '/') {
    response.writeHead(200, { 'Content-Type': 'text/html; charset=utf-8' });
    response.end(page);
  } else {
    response.writeHead(404);
    response.end('Not found');
  }
}).listen(3000, '127.0.0.1', () => console.log('Open http://127.0.0.1:3000'));
```

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<title>Live events</title>
<h1>Server clock</h1>
<p id="connection" role="status">Connecting…</p>
<p>Latest server time: <time id="clock">Waiting</time></p>
<button id="stop" type="button">Stop updates</button>
<script>
  const status = document.querySelector('#connection');
  const clock = document.querySelector('#clock');
  let events;
  let stopped = false;
  function disconnect() { events?.close(); events = undefined; }
  function connect() {
    disconnect();
    status.textContent = 'Connecting…';
    const source = new EventSource('/events');
    events = source;
    source.addEventListener('open', () => {
      if (events === source) status.textContent = 'Connected';
    });
    source.addEventListener('tick', event => {
      if (events !== source) return;
      try {
        const payload = JSON.parse(event.data);
        if (typeof payload.time !== 'string') throw new Error('Invalid tick');
        clock.textContent = payload.time;
        clock.dateTime = payload.time;
      } catch (error) {
        status.textContent = 'Received an invalid update.';
        console.error(error);
      }
    });
    source.addEventListener('error', () => {
      if (events === source) status.textContent = 'Connection interrupted; reconnecting…';
    });
  }
  document.querySelector('#stop').addEventListener('click', () => {
    stopped = true;
    disconnect();
    status.textContent = 'Updates stopped';
  });
  addEventListener('pagehide', disconnect);
  addEventListener('pageshow', event => {
    if (event.persisted && !stopped) connect();
  });
  connect();
</script>
</html>
```

Run `node server.mjs` and open the printed URL. Expect a new timestamp approximately every second. Stop the server, observe the reconnecting state, restart it, and confirm updates resume. Navigate away and use Back: a page restored from the back/forward cache reconnects. Stop updates closes the connection and preserves that choice across restoration. The clock itself is not a live region, avoiding a screen-reader announcement every second.

This fixture resets IDs per connection and does not replay events; it demonstrates streaming and cleanup only. Its periodic writes also omit a production backpressure policy. Do not interpret the presence of an `id` field as reliable delivery.

## When WebSockets fit

A collaborative editor, multiplayer interaction, or bidirectional command channel may need WebSockets. The browser API exposes open, message, error, and close events plus `send()`. Send only after the connection is open, validate incoming messages, and close the connection when its owner disappears. Check `bufferedAmount` and define a queue limit rather than allowing unbounded pending writes. The classic browser WebSocket API does not provide automatic backpressure; see the [WebSocket reference](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket).

Define an application protocol with message type, version, operation ID, and payload. Heartbeats can detect dead connections, while exponential reconnect backoff with jitter reduces reconnect storms. A successful send does not prove the server applied an operation; consequential commands need acknowledgments and idempotent handling.

## Deployment and correctness

Proxies may buffer SSE or impose idle timeouts. Configure streaming deliberately and send appropriate heartbeat comments when needed. Authenticate connections and authorize each subscribed resource. Browser EventSource does not offer arbitrary custom request headers, so choose a supported authentication design rather than assuming it behaves exactly like Fetch.

Events may arrive twice, late, or out of order across reconnects and distributed producers. Use stable IDs and sequence or revision information, deduplicate where required, and resynchronize after gaps. Background tabs and mobile network changes also affect timing; UI timestamps should communicate freshness instead of implying perfect continuity.

## Exercises and acceptance criteria

1. Add a connection-scoped heartbeat comment. Acceptance: the client ignores it while proxies still receive traffic.
2. Add retained event history and resume from a last event ID. Acceptance: reconnect recovers a missed event exactly once in the rendered view.
3. Design a WebSocket message contract for chat. Acceptance: include acknowledgment, duplicate handling, reconnect behavior, authorization, and a bounded outgoing queue.

## References

- [WHATWG: server-sent events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [MDN: writing WebSocket clients](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API/Writing_WebSocket_client_applications)
