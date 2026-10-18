# 62. Browser APIs and permissions

[Curriculum](../README.md)

**Prerequisites:** [DOM events](../17-dom-events-and-interaction/tutorial.md), [async JavaScript](../18-promises-async-and-the-event-loop/tutorial.md), and [progressive enhancement](../70-compatibility-and-progressive-enhancement/tutorial.md).

**You will learn to:** request a capability in context, handle denial, release resources, and provide useful fallbacks.

## Capabilities are conditional

Browser APIs expose location, clipboard access, notifications, device media, screen sharing, fullscreen, sharing, and more. Availability depends on the browser, operating system, origin security, embedding policy, permissions, and user activation. Detecting a method proves only that an entry point exists; the operation may still fail.

Ask for access when the user chooses an action with an understandable purpose. Explain what is needed before the prompt, but do not show repeated prompts after refusal. Provide a manual path when possible: entering a location, selecting a file, or copying selected text. Store a user's preference separately from the browser's permission state because either may change independently.

The Permissions API can inspect some permission states but is not a universal permission-request mechanism. Some capabilities are gated by a temporary user gesture. Calling an API after a long asynchronous chain can lose that activation. Embedded applications may also need an appropriate Permissions Policy and iframe `allow` configuration.

## Tutorial: a camera preview with cleanup

Create `index.html` and serve it on localhost. Camera access generally requires a secure context and permission; see [`getUserMedia`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia).

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Camera preview</title>
<h1>Preview your camera</h1>
<p>The preview stays on this page. This example does not upload video.</p>
<button id="start">Start camera</button>
<button id="stop" disabled>Stop camera</button>
<p id="status" role="status"></p>
<video id="preview" autoplay muted playsinline style="max-width:100%"></video>
<script type="module">
const start = document.querySelector('#start');
const stop = document.querySelector('#stop');
const status = document.querySelector('#status');
const preview = document.querySelector('#preview');
let stream;
let generation = 0;

function release() {
  generation++;
  stream?.getTracks().forEach(track => track.stop());
  stream = undefined;
  preview.srcObject = null;
  start.disabled = false;
  stop.disabled = true;
  status.textContent = 'Camera stopped.';
}

start.addEventListener('click', async () => {
  if (!navigator.mediaDevices?.getUserMedia) {
    status.textContent = 'Camera access is unavailable here.';
    return;
  }
  const requestId = ++generation;
  start.disabled = true;
  stop.disabled = false;
  status.textContent = 'Waiting for camera permission…';
  try {
    const acquired = await navigator.mediaDevices.getUserMedia({ video: true, audio: false });
    if (requestId !== generation) {
      acquired.getTracks().forEach(track => track.stop());
      return;
    }
    stream = acquired;
    preview.srcObject = stream;
    await preview.play();
    if (requestId === generation) status.textContent = 'Camera is active.';
  } catch (error) {
    if (requestId !== generation) return;
    release();
    status.textContent = `Could not start camera: ${error.name}.`;
  }
});
stop.addEventListener('click', release);
window.addEventListener('pagehide', release);
</script>
</html>
```

Allow access: the video should appear. Stop it: the tracks should end. Cancel while permission is pending: if the browser later resolves the request, the generation check immediately stops the stale stream. That check prevents a resource leak when a request outlives the user's intention.

## Other API families

Use Intersection Observer for visibility thresholds, Resize Observer for element-size changes, and Mutation Observer only when DOM mutation itself is the event of interest. Each observer needs cleanup when its owner disappears. Avoid resize feedback loops that continuously change the measured element.

Geolocation provides potentially sensitive and imperfect coordinates; explain accuracy and allow correction. Clipboard read access is more constrained than writing. Notifications and push require careful timing, service-worker coordination, and a clear unsubscribe experience. The Web Share API should fall back to normal links or copying. Device-specific APIs such as Bluetooth, USB, serial, MIDI, and NFC require capability checks and a browser-support decision before product design depends on them.

## Debug and practice

Test permission allowed, denied, pending, revoked, and unavailable. Also test no camera, camera in use, a hidden tab, navigation away, and an iframe deployment. Distinguish permission denial from unsupported constraints or hardware failure in actionable user messages. Do not infer a permanent refusal from every exception.

**Exercise:** add camera selection and a still-image capture action. Acceptance criteria: a previous stream stops before switching, late requests cannot overwrite the newest selection, all tracks stop on exit, and users without camera access can choose an image file instead.

**References:** [Permissions API](https://developer.mozilla.org/en-US/docs/Web/API/Permissions_API), [Secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts), [Intersection Observer](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).
