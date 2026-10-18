# Setup and example conventions

[Curriculum](README.md)

## What you need

Use a code editor, an up-to-date browser with DevTools, and a terminal. Install Git for version-control exercises. Install a supported Node.js release compatible with your selected tooling for package and framework exercises. The documentation checks for this library ran with Node 24.19.0 and npm 11.17.0; this does not mean every framework example was installed under those versions.

Verify your environment:

```powershell
node --version
npm --version
git --version
```

On Windows, if PowerShell blocks `npm.ps1`, use `npm.cmd` in the same commands. Do not disable machine-wide security settings merely to run a tutorial. If `python` opens an installation prompt rather than reporting a version, use a Node-based server or install Python intentionally.

## Keep practice separate from the tutorials

Create a separate `frontend-practice` folder alongside this library or elsewhere you own. Give each exercise its own subfolder. Do not run every framework scaffold in the curriculum directory: they create application files that are unrelated to the Markdown library.

For a plain browser lesson, create `index.html` and the CSS/JavaScript files named by the lesson. Use this shell when a chapter provides only body content:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Frontend exercise</title>
  </head>
  <body>
    <!-- Insert the chapter's body markup here. -->
    <!-- Add its script or script reference in the stated order. -->
  </body>
</html>
```

A complete HTML document should be used as supplied, not nested inside another document. A body fragment is deliberately partial. CSS belongs in a stylesheet linked from the document or in a `<style>` element. Browser JavaScript belongs in a script with the appropriate module setting. Node scripts do not have a browser DOM.

## Serve browser exercises locally

From the exercise folder, one option is:

```powershell
npx --yes http-server@14.1.1 . -a 127.0.0.1 -p 8080 -c-1
```

This downloads and executes the named development-server package if it is not already cached. Use it only for local practice; it is not a production server. Open `http://127.0.0.1:8080` and leave the terminal running. Stop it with Ctrl+C. If the port is busy, choose another port and update your browser URL. Alternatively, if Python is installed, run `python -m http.server 8080 --bind 127.0.0.1`.

Use HTTP on loopback rather than opening files with `file://`: modules, fetch, workers, and origin rules behave differently. Many secure-context APIs accept loopback development origins, but device and permission requirements still apply. A real deployment generally needs HTTPS. Accessing a development server from a phone is a different setup and may require local-network binding and HTTPS.

## Framework exercises

Follow the chapter's scaffold command and inspect the generated package manifest. Installation needs network access. Framework tools may ask interactive questions or change generated filenames. Keep the selected framework major and its official guide consistent. When a chapter names a component file, adapt the generated import path if your scaffold uses another filename.

Run commands from the directory containing the intended `package.json`. `npm install` resolves dependencies and creates or updates the lockfile. `npm ci` requires a compatible existing lockfile and is appropriate for repeatable clean installs. Do not use force flags as the first response to dependency conflicts.

When an example says `/api/products`, it assumes a server route or explicit development proxy described by that lesson. A relative URL on a static server does not create an API. Mocked data, illustrative server contracts, and real network requests are identified separately in the chapters.

## Read examples accurately

- **Complete example:** all essential code for the described small demo is supplied.
- **Body fragment:** insert into the HTML shell above.
- **Application excerpt:** requires the stated framework, imports, surrounding component, or server endpoint.
- **Pseudocode or protocol sketch:** explains a design and is not a copy-and-run application.
- **Intentional failure:** demonstrates a bug or a failing test before a later correction.

Use DevTools Console, Network, Elements, Accessibility, Application/Storage, and Performance panels as relevant. Check the terminal as well as the browser. Preserve the first error message; later errors can be consequences rather than causes.

## Working habits

Commit a working baseline before large experiments. Keep dependencies and lockfiles together. Do not put passwords, service keys, or real personal records into exercises. Use generated demo data. When a tutorial needs a backend, a media asset, or a service account, create only the resources you intend to use and understand their lifecycle.

After each lesson, reproduce the example without copying, explain why it works, and complete at least one exercise. Being able to identify a failure mode is as valuable as obtaining the expected screenshot.
