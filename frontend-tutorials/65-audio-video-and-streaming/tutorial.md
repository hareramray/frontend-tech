# 65. Audio, video, and streaming

[Curriculum](../README.md)

**Prerequisites:** [media assets](../10-images-svg-and-media-assets/tutorial.md), [browser permissions](../62-browser-apis-and-permissions/tutorial.md), and [accessibility](../05-web-accessibility/tutorial.md).

**You will learn to:** use native media controls, generate a short sound safely, and distinguish playback, processing, and delivery protocols.

## Start with the media experience

A media feature includes more than a player. Decide what users need before playback, while buffering, after an error, and when audio is unavailable. Use captions for speech and important sounds, transcripts where appropriate, and audio descriptions when essential visual information is absent from the audio. Do not make sound the only signal for an alert.

An HTML `<video controls>` or `<audio controls>` is a useful starting point. Custom controls must reproduce keyboard access, names, state announcements, volume, seeking, fullscreen where supported, and error behavior. Native controls reduce the amount of interaction you must implement and test.

Containers and codecs are different: an MP4 or WebM container can carry particular encoded tracks. File extension alone does not guarantee playback support. Test actual codec/profile combinations on target devices, and supply alternatives when required. Adaptive streaming, latency, bandwidth, battery use, and content protection create additional constraints.

## Tutorial: generate a short confirmation tone

This standalone example needs no downloaded media. Create `index.html` and serve it locally:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Audio feedback</title>
<h1>Sound preview</h1>
<button id="play">Play a quiet tone</button>
<p id="status" role="status"></p>
<script type="module">
const button = document.querySelector('#play');
const status = document.querySelector('#status');
let context;
button.addEventListener('click', async () => {
  button.disabled = true;
  try {
    const AudioContextClass = window.AudioContext;
    if (!AudioContextClass) throw new Error('Web Audio is unavailable');
    context ??= new AudioContextClass();
    await context.resume();
    const tone = context.createOscillator();
    const gain = context.createGain();
    const now = context.currentTime;
    tone.frequency.value = 440;
    gain.gain.setValueAtTime(0, now);
    gain.gain.linearRampToValueAtTime(0.08, now + 0.02);
    gain.gain.linearRampToValueAtTime(0, now + 0.22);
    tone.connect(gain).connect(context.destination);
    tone.onended = () => {
      tone.disconnect();
      gain.disconnect();
      button.disabled = false;
      status.textContent = 'Tone finished.';
    };
    tone.start(now);
    tone.stop(now + 0.25);
    status.textContent = 'Playing a short tone.';
  } catch (error) {
    status.textContent = error.message;
    button.disabled = false;
  }
});
window.addEventListener('pagehide', () => {
  const oldContext = context;
  context = undefined;
  button.disabled = false;
  if (oldContext && oldContext.state !== 'closed') oldContext.close().catch(() => {});
});
</script>
</html>
```

Clicking plays a short 440 Hz tone at a deliberately low gain and displays a text status. The gain ramps reduce clicks caused by abrupt waveform changes. An oscillator is a single-use source; create another for the next playback. Web Audio connects sources through processing nodes to a destination, as described in the [Web Audio guide](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Using_Web_Audio_API).

## Extend to recorded media

For a real video, add a poster, explicit dimensions or aspect ratio, `controls`, `playsinline`, a conservative preload strategy, and a `<track kind="captions" ...>` referencing a WebVTT file. A poster should not misrepresent the content. Keep caption language and labels accurate. Call `play()` from an appropriate user action and handle its rejected promise instead of assuming autoplay succeeds.

Media Source Extensions let code feed encoded segments into a media element. HLS and MPEG-DASH describe segmented adaptive delivery; browser-native support and JavaScript player requirements vary. Adaptive bitrate selection chooses among encodings based on conditions. Live streaming adds a sliding window and latency tradeoffs; a live stream is not necessarily a real-time conversation.

WebCodecs exposes lower-level encode/decode primitives in supporting environments. It does not by itself implement container parsing, network transport, or an accessible player. AudioWorklet supports custom low-latency audio processing without placing that processing loop on the ordinary main thread. Avoid allocation-heavy work in time-critical audio processing.

## Debug and practice

Inspect network range requests, MIME types, CORS headers, buffering events, and decode errors. A successful download does not prove a device can decode the selected media. Cross-origin media may need CORS configuration for canvas extraction or audio processing. Release streams and object URLs, and stop playback intentionally when an owning component disappears.

**Exercise:** build an accessible player for a short video you own. Acceptance criteria: captions and transcript are available, the controls work by keyboard, playback failure has a visible message, no unsolicited sound starts, and a low-bandwidth test demonstrates useful loading feedback.

**References:** [Audio and video delivery](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Audio_and_video_delivery), [Media Source API](https://developer.mozilla.org/en-US/docs/Web/API/Media_Source_Extensions_API), [WebVTT](https://developer.mozilla.org/en-US/docs/Web/API/WebVTT_API).
