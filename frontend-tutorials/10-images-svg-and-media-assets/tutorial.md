# Images, SVG, and media assets

[Curriculum](../README.md)

## Prerequisites and outcomes

Read [semantic HTML](../03-semantic-html/tutorial.md), [responsive design](../08-responsive-design/tutorial.md), and [accessibility](../05-web-accessibility/tutorial.md). You will choose asset formats, preserve image geometry, write useful alternatives, create a local SVG, and plan responsive delivery without unnecessarily increasing page weight.

## Choose a format for the content

Raster images store pixels and suit photographs, textured illustrations, and screenshots. JPEG is widely useful for photographs without transparency; PNG preserves lossless detail and transparency; WebP and AVIF offer additional compression options with different encoding and decoding tradeoffs. Compare actual files at the intended display size instead of assuming a format always wins. SVG describes vector geometry and is useful for icons, diagrams, and logos, but complex SVG can be large or expensive to render.

CSS controls displayed dimensions; it does not necessarily control downloaded bytes. `srcset` offers image candidates and `sizes` describes the expected rendered slot for width-descriptor candidates. The browser chooses using layout hints, pixel density, and its own selection logic. A `2x` density candidate and a `640w` width candidate use different descriptor systems and should not be mixed in one candidate list. `picture` supports art direction or format alternatives; its final `img` remains the actual image and supplies alternative text.

Give images intrinsic `width` and `height` where known so browsers can reserve the right aspect ratio before decoding. Keep CSS responsive with `max-inline-size: 100%` and `block-size: auto`. `object-fit: cover` crops within a constrained box; it does not resize the source file or guarantee that a person's face remains visible. Use `object-position` or a deliberately composed alternative when cropping matters.

Alternative text depends on purpose. Describe meaningful information, not merely the filename or the phrase “image of.” Decorative images use `alt=""`; functional images need an alternative describing the action or destination. Complex charts need a fuller explanation or equivalent data. See [WAI's image decision tree](https://www.w3.org/WAI/tutorials/images/decision-tree/).

## Worked tutorial: a local illustrated card

1. Create a folder with `index.html` and `landscape.svg`. Save this SVG as `landscape.svg`:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 800 450">
  <rect width="800" height="450" fill="#dcecff"/>
  <circle cx="620" cy="95" r="48" fill="#ffd36b"/>
  <path d="M0 380 220 120 430 380Z" fill="#527c83"/>
  <path d="M260 450 540 170 800 450Z" fill="#315c6b"/>
  <path d="M170 180 220 120 270 182 225 166Z" fill="#fff"/>
</svg>
```

2. Save the HTML below, then open it directly in a browser. No downloaded assets are needed.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Media card</title>
<style>
  * { box-sizing: border-box; }
  body { margin: 2rem 1rem; font: 1.1rem/1.6 system-ui; color: #18243b; }
  main { max-inline-size: 42rem; margin-inline: auto; }
  figure { margin: 0; }
  img { display: block; max-inline-size: 100%; block-size: auto; }
  figcaption { padding-block: .75rem; }
  .icon { inline-size: 1.25em; block-size: 1.25em; vertical-align: -.2em; }
  a { color: #174ea6; }
</style>
<main>
  <h1>Mountain illustration</h1>
  <figure>
    <img src="landscape.svg" width="800" height="450"
      alt="Two triangular mountain peaks beneath a yellow sun in a pale blue sky.">
    <figcaption>A locally authored vector illustration that scales with the page.</figcaption>
  </figure>
  <p>
    <a href="landscape.svg" download>
      <svg class="icon" viewBox="0 0 24 24" aria-hidden="true" focusable="false">
        <path d="M12 3v12m-5-5 5 5 5-5M4 18v3h16v-3"
          fill="none" stroke="currentColor" stroke-width="2"/>
      </svg>
      Download illustration
    </a>
  </p>
</main>
</html>
```

3. Resize the page and inspect its aspect ratio. The image remains 16:9 without becoming wider than its container.
4. Disable image loading or temporarily rename the SVG. The alternative text should communicate the image's subject. Restore the filename.

The inline download icon is decorative because visible link text already communicates the action. `currentColor` keeps its stroke aligned with the link's color. The external image is informative, so it has its own alternative. Inline SVG can expose richer semantics, but inserted SVG from untrusted sources requires careful sanitization because SVG can contain active or externally referenced content.

## Delivery, video, and advanced tradeoffs

For a photographic version, export several real widths and use a candidate list such as `photo-480.webp 480w, photo-960.webp 960w` with a `sizes` value matching your layout. Do not label a 480-pixel file as `960w`; inaccurate hints waste bandwidth or reduce quality. Inspect the chosen `currentSrc` and actual transferred bytes at different widths and pixel densities.

Lazy loading is useful for offscreen images but can delay an important initially visible image. Do not indiscriminately apply `loading="lazy"` to the hero. Consider `fetchpriority` only for resources whose priority you have measured. `decoding` is a hint, not a guarantee of when pixels appear. The [MDN image reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/img) documents these attributes.

Video needs controls, appropriate preload behavior, captions through a WebVTT track, and alternatives such as a transcript or audio description when the content requires them. Audio autoplay is often restricted and can disrupt users. A poster should accurately represent the content. For streaming, choose delivery based on duration, bandwidth, seek needs, and supported formats rather than treating a giant MP4 as the only option.

Canvas can process raster data, but cross-origin images without appropriate CORS permission can taint the canvas and prevent readback. Preserve original assets, track licenses and attribution, remove unintended metadata, and automate compression while visually reviewing the result.

## Exercises and acceptance criteria

- Add a decorative duplicate image with empty alternative text. Explain why its purpose differs from the original.
- Export a photograph at two actual widths and build `srcset`/`sizes`. Network inspection must confirm sensible selection at narrow and wide layouts.
- Place the image in a square crop. Adjust `object-position` and document what information is lost.
- Write a media checklist covering dimensions, bytes, alternatives, licensing, and loading priority.

## References

- [MDN: responsive images](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Responsive_images)
- [W3C WAI: images tutorial](https://www.w3.org/WAI/tutorials/images/)
- [MDN: SVG tutorial](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial)
- [MDN: video element](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video)
