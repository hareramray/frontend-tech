# Project 01. Build an accessible portfolio

[Projects](../README.md) · [Curriculum](../../README.md)

**Prerequisites:** chapters 01–12, [DOM events](../../17-dom-events-and-interaction/tutorial.md), and [accessibility testing](../../49-accessibility-testing/tutorial.md).

**Outcome:** a small portfolio with a home page, project detail page, and contact page. It works on narrow screens, supports keyboard navigation, and communicates useful information without optional JavaScript.

## Define the content first

Choose a fictional or personal profile and three projects you can describe accurately. For each project, write the problem, your contribution, constraints, result, and a link or screenshot you have permission to use. Avoid placeholder paragraphs that hide whether the layout handles real content. Write the contact method before styling a form; a form needs a real submission destination to send anything.

Create `index.html`, `project.html`, `contact.html`, `styles.css`, and an `assets/` directory. Serve the folder using the [setup guide](../../SETUP.md). Start with text and links before adding decoration.

## Milestone 1: semantic skeleton

Use this as the home page's body inside a normal HTML shell:

```html
<a class="skip" href="#main">Skip to content</a>
<header>
  <a href="index.html">Mira Rao — frontend developer</a>
  <nav aria-label="Primary">
    <ul>
      <li><a href="index.html" aria-current="page">Work</a></li>
      <li><a href="contact.html">Contact</a></li>
    </ul>
  </nav>
</header>
<main id="main" tabindex="-1">
  <h1>Interfaces that help people complete real tasks</h1>
  <p>I build accessible tools for learning and everyday work.</p>
  <section aria-labelledby="work-title">
    <h2 id="work-title">Selected work</h2>
    <div class="projects">
      <article>
        <h3><a href="project.html">Community reading catalog</a></h3>
        <p>A searchable catalog designed for keyboard and mobile use.</p>
      </article>
      <article>
        <h3>Local events guide</h3>
        <p>A responsive guide with readable schedules and clear locations.</p>
      </article>
    </div>
  </section>
</main>
<footer><p>© 2026 Mira Rao</p></footer>
```

Replace fictional details before publishing as your own portfolio. On each page, set a unique title and heading, and move `aria-current="page"` to the correct navigation item. Do not give every card a link until the destination exists.

**Check:** headings form a coherent outline, links have meaningful text, the skip link reaches main content, and navigation works with JavaScript disabled.

## Milestone 2: responsive layout

Link `styles.css` from the head and start with:

```css
* { box-sizing: border-box; }
body { margin: 0; color: #17202a; background: #fff; font: 1rem/1.6 system-ui; }
header, main, footer { width: min(100% - 2rem, 68rem); margin-inline: auto; }
header { display: flex; flex-wrap: wrap; align-items: center; justify-content: space-between; gap: 1rem; padding-block: 1rem; }
nav ul { display: flex; flex-wrap: wrap; gap: 1rem; padding: 0; list-style: none; }
.projects { display: grid; grid-template-columns: repeat(auto-fit, minmax(min(100%, 18rem), 1fr)); gap: 1.5rem; }
article { border: 1px solid #687582; border-radius: .5rem; padding: 1.25rem; }
p { max-width: 65ch; }
a { color: #174f9c; text-underline-offset: .2em; }
:focus-visible { outline: 3px solid #913500; outline-offset: 4px; }
.skip { position: absolute; left: 1rem; top: 0; transform: translateY(-150%); background: white; padding: .75rem; }
.skip:focus { transform: translateY(0); }
img { max-width: 100%; height: auto; }
```

**Check:** at a narrow viewport there is no page-wide horizontal scroll. Long project titles wrap. At browser zoom, controls remain usable. Do not “fix” overflow by hiding all horizontal content; locate the element causing it.

## Milestone 3: project detail and contact

Build the detail page with a problem statement, constraints, implementation explanation, and results. Add images with intrinsic dimensions, appropriate alt text, and responsive sources when you have multiple sizes. Decorative images use empty alternative text. Do not lazy-load a prominent image blindly if it is the page's main visible content.

For contact, begin with a working email link and a clearly stated response expectation. If you add a form, implement or configure its server destination, server-side validation, rate limiting, and accessible success/failure feedback. Do not show “message sent” merely because a local button was clicked. A client-only demonstration must say that it is a demonstration.

## Milestone 4: verify and publish

Check every link, page title, image path, and active navigation state. Walk through all pages by keyboard, then use a screen reader for a short navigation pass. Run an automated accessibility check and investigate findings. Test with images disabled and with a slow network. Inspect layout shift and oversized images.

Deploy the static files to a host of your choice, then repeat deep-link and asset checks on the real URL. Verify that the contact destination actually receives a test message if a form exists. Keep a small review document describing the tests and any remaining limitations.

**Completion criteria:** all three pages exist; content remains understandable without scripts; focus is visible; headings and landmarks are meaningful; narrow-screen layout is usable; images have correct alternatives; contact behavior is truthful; and the deployed site has no broken internal links.

**Stretch work:** add a theme switch using CSS custom properties, preserve the preference safely, provide a print stylesheet, and test reduced motion if you introduce animation. Avoid adding a framework unless it solves a concrete need in this project.
