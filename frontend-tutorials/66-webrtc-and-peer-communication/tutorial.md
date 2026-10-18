# 66. WebRTC and peer communication

[Curriculum](../README.md)

**Prerequisites:** [real-time transport](../37-realtime-websockets-and-sse/tutorial.md), [permissions](../62-browser-apis-and-permissions/tutorial.md), and [async JavaScript](../18-promises-async-and-the-event-loop/tutorial.md).

**You will learn to:** explain signaling and ICE, establish a local data channel, and identify the infrastructure required for real calls.

## Three responsibilities

WebRTC provides browser APIs for real-time media and data exchange. The application still needs signaling: a way for participants to exchange session descriptions and candidates. Signaling can use WebSockets or another authenticated channel; WebRTC does not prescribe your room model or user identity.

Session Description Protocol describes negotiated capabilities. ICE finds a viable connectivity path. STUN helps discover reachable addressing information; TURN relays traffic when a direct path cannot be established. A production system needs a TURN strategy because two devices on different networks cannot be assumed to connect directly. Signaling and negotiation are explained in MDN's [signaling guide](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Signaling_and_video_calling).

Transport encryption does not prove that the person joining a room is authorized. Authentication, room access, abuse controls, and participant identity belong to the application. A relay or media server changes privacy and operational considerations; explain the actual architecture accurately.

## Tutorial: connect two peers in one page

This lab demonstrates negotiation without a signaling server. It is not an internet calling service. Create `index.html` with a `<button id="connect">Connect local peers</button>`, a `<pre id="log" role="status"></pre>`, and a module script containing the following. Serve it at localhost.

```js
const button = document.querySelector('#connect');
const log = document.querySelector('#log');
let left;
let right;

function waitForIce(peer) {
  if (peer.iceGatheringState === 'complete') return Promise.resolve();
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(() => {
      peer.removeEventListener('icegatheringstatechange', check);
      reject(new Error('ICE gathering timed out'));
    }, 10000);
    function check() {
      if (peer.iceGatheringState !== 'complete') return;
      clearTimeout(timeout);
      peer.removeEventListener('icegatheringstatechange', check);
      resolve();
    }
    peer.addEventListener('icegatheringstatechange', check);
    check();
  });
}

function closePeers() {
  left?.close();
  right?.close();
}

button.addEventListener('click', async () => {
  button.disabled = true;
  closePeers();
  try {
    left = new RTCPeerConnection();
    right = new RTCPeerConnection();
    right.ondatachannel = ({ channel }) => {
      channel.onmessage = ({ data }) => {
        log.textContent = `Received: ${data}`;
        closePeers();
        button.disabled = false;
      };
    };
    const channel = left.createDataChannel('course');
    channel.onopen = () => channel.send('Hello from the other peer');
    await left.setLocalDescription(await left.createOffer());
    await waitForIce(left);
    await right.setRemoteDescription(left.localDescription);
    await right.setLocalDescription(await right.createAnswer());
    await waitForIce(right);
    await left.setRemoteDescription(right.localDescription);
    log.textContent = 'Negotiation complete; waiting for the channel…';
  } catch (error) {
    log.textContent = error.message;
    closePeers();
    button.disabled = false;
  }
});
window.addEventListener('pagehide', closePeers);
```

Expected result: the receiver displays the message and both connections close. If gathering completes but the channel does not open, inspect connection and ICE states. Add a connection deadline in a production implementation; this teaching example only times out gathering. The lab waits for gathered candidates before exchanging full descriptions. Production applications commonly send candidates incrementally, known as trickle ICE.

## From a lab to a real call

Replace the direct description assignments with authenticated signaling messages. Validate message shape and room membership on the server. Queue remote candidates until a remote description exists. Handle simultaneous offers with an established negotiation pattern instead of assuming only one participant can initiate changes.

For media, obtain a stream after user action, add its tracks to the connection, and attach incoming tracks to remote media elements. Support mute, device changes, ended tracks, and explicit hang-up. Stopping a sender and stopping the physical capture track are different operations. Release both connection and device resources.

For multiparty calls, a full mesh scales poorly because each participant sends multiple copies. An SFU forwards selected streams; an MCU mixes media. Simulcast, scalable video coding, congestion control, echo cancellation, and jitter buffering affect quality. Plan bandwidth, relay cost, and reconnect behavior as part of the feature.

Data channels support configurable delivery behavior, but a channel is not unlimited storage. Check `bufferedAmount` and use backpressure for large transfers. Chunk files and verify completeness. Real-time state should tolerate delayed or missing updates when using unreliable delivery.

**Exercise:** split the lab into two pages connected through your own development signaling server. Acceptance criteria: only authorized room members exchange messages, a timeout is visible, hang-up releases resources, and the design documents a TURN-backed test across different networks.

**References:** [WebRTC perfect negotiation](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/Perfect_negotiation), [RTCDataChannel](https://developer.mozilla.org/en-US/docs/Web/API/RTCDataChannel), [WebRTC samples](https://webrtc.github.io/samples/).
