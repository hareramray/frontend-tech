# 64. Canvas, WebGL, and WebGPU

[Curriculum](../README.md)

**Prerequisites:** [DOM](../17-dom-events-and-interaction/tutorial.md), [responsive design](../08-responsive-design/tutorial.md), and [performance](../50-web-performance-and-core-web-vitals/tutorial.md).

**You will learn to:** choose a graphics representation, draw a crisp canvas, and understand the GPU pipeline and its constraints.

## Choose a representation

HTML provides semantics and ordinary layout. SVG describes retained vector objects in the DOM and works well for icons, diagrams, and moderate interactive charts. Canvas 2D draws pixels into a bitmap; your application owns scene state, hit testing, redraws, and accessibility. WebGL and WebGPU expose programmable GPU pipelines for demanding rendering or computation. A technology with more power can also create more maintenance work.

For charts, preserve a textual summary and accessible data representation. A label on a canvas is not a substitute for all the information drawn inside it. For editing tools, use HTML controls around the drawing surface and supply keyboard equivalents to pointer operations.

## Tutorial: draw a responsive bar chart

Create `index.html`, serve it locally, and open it at several viewport widths:

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Canvas chart</title>
<h1>Weekly completed tasks</h1>
<p id="summary">Monday: 4. Tuesday: 7. Wednesday: 5.</p>
<canvas aria-describedby="summary" role="img" aria-label="Bar chart of completed tasks"
  style="width:100%;max-width:640px;height:240px;display:block"></canvas>
<script type="module">
const canvas = document.querySelector('canvas');
const context = canvas.getContext('2d');
const values = [4, 7, 5];
const labels = ['Mon', 'Tue', 'Wed'];
function draw() {
  if (!context) return;
  const width = canvas.clientWidth;
  const height = canvas.clientHeight;
  const ratio = Math.min(window.devicePixelRatio || 1, 2);
  canvas.width = Math.round(width * ratio);
  canvas.height = Math.round(height * ratio);
  context.setTransform(ratio, 0, 0, ratio, 0, 0);
  context.clearRect(0, 0, width, height);
  const slot = width / values.length;
  const baseline = height - 32;
  context.font = '16px system-ui';
  context.textAlign = 'center';
  values.forEach((value, index) => {
    const barHeight = value / 10 * (height - 60);
    const x = slot * index + slot * 0.2;
    context.fillStyle = '#1757a6';
    context.fillRect(x, baseline - barHeight, slot * 0.6, barHeight);
    context.fillStyle = '#17202a';
    context.fillText(labels[index], slot * (index + 0.5), height - 8);
    context.fillText(String(value), slot * (index + 0.5), baseline - barHeight - 7);
  });
}
const observer = new ResizeObserver(draw);
observer.observe(canvas);
window.addEventListener('resize', draw);
draw();
</script>
</html>
```

Expected result: three labeled bars remain legible when resized. CSS controls display size; canvas width and height control backing-store resolution. Assigning those properties resets drawing state, so reapply transforms and styles afterward. The pixel-ratio cap trades sharpness on very dense screens for memory and fill-rate savings. In a component, disconnect the observer and remove the resize listener when it unmounts.

## Build a graphics mental model

Canvas redraws are immediate: a later fill can cover earlier content. Store scene objects separately so resizing or undo can reconstruct the image. Convert pointer coordinates using the canvas bounding rectangle; account for scaling and transforms. For animation, compute movement from elapsed time, clamp extreme time gaps, pause offscreen work, and respect reduced motion.

In a GPU pipeline, vertex data defines geometry, a vertex shader transforms positions, rasterization creates fragments, and a fragment shader computes their color. Textures supply sampled data; uniforms or buffers supply parameters. Coordinate spaces, matrix order, blending, depth testing, and color management matter more than the library's convenience methods.

WebGL uses GLSL shaders and an established browser API. WebGPU uses explicit resources and pipelines, WGSL shaders, and compute capabilities. Do not assume availability: feature-detect `navigator.gpu`, handle a null adapter, device-request failure, and device loss. Secure context and implementation support remain deployment considerations. See the [WebGPU API overview](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API).

## Performance and failure handling

Measure CPU time and GPU load separately. Reduce draw calls, reuse buffers, batch similar objects, compress textures when appropriate, and avoid reading pixels back every frame. Textures consume memory beyond the compressed download size. Dispose of library-managed materials, textures, and geometry when they are no longer needed. Handle context loss with a recovery path or a clear fallback.

OffscreenCanvas can move some rendering into a worker, subject to API support and ownership constraints. Three.js and Babylon.js provide scene abstractions, but do not remove the need to understand asset size, lighting cost, accessible alternatives, or lifecycle cleanup.

**Exercise:** add a toggle between the canvas chart and an HTML table. Acceptance criteria: both expose identical values, keyboard users can switch views, the chart stays crisp after resize, zero and empty data have sensible output, and repeated mounting does not accumulate observers or graphics resources.

**References:** [Canvas tutorial](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial), [WebGL best practices](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API/WebGL_best_practices), [Three.js documentation](https://threejs.org/docs/).
