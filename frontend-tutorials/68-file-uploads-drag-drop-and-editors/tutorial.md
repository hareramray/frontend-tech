# 68. File uploads, drag and drop, and editors

[Curriculum](../README.md)

**Prerequisites:** [forms](../04-html-forms-and-validation/tutorial.md), [fetch](../34-http-fetch-and-rest/tutorial.md), and [security](../40-frontend-security-and-privacy/tutorial.md).

**You will learn to:** handle a selected file, provide an accessible alternative to dropping, and design safe upload and editing workflows.

## Treat file handling as a pipeline

Selection, validation, preview, upload, processing, and publication are separate states. A local preview does not mean a file has been uploaded. A completed upload does not mean the server has accepted or processed the file. Model these states explicitly so users know whether it is safe to navigate away.

`accept` on a file input guides selection but is not security validation. Filenames and MIME types supplied by clients are untrusted. The server must enforce authorization, size limits, actual content checks, storage policy, and safe download behavior. Never execute a file because its extension looks harmless.

## Tutorial: preview a small text file

Create this body fragment inside the [standard HTML shell](../SETUP.md) and serve it locally:

```html
<h1>Read a local note</h1>
<label for="file">Choose a text file, up to 100 KB</label>
<input id="file" type="file" accept=".txt,text/plain">
<div id="drop" style="padding:2rem;border:2px dashed">Or drop a file here.</div>
<p id="status" role="status"></p>
<pre id="preview" style="white-space:pre-wrap;overflow-wrap:anywhere"></pre>
<script type="module">
const input = document.querySelector('#file');
const drop = document.querySelector('#drop');
const status = document.querySelector('#status');
const preview = document.querySelector('#preview');
let revision = 0;
async function show(file) {
  const current = ++revision;
  preview.textContent = '';
  if (!file) return;
  if (file.size > 100000) {
    status.textContent = 'Choose a file no larger than 100 KB.';
    return;
  }
  if (!file.name.toLowerCase().endsWith('.txt')) {
    status.textContent = 'Choose a .txt file for this demo.';
    return;
  }
  status.textContent = 'Reading…';
  try {
    const text = await file.text();
    if (current !== revision) return;
    preview.textContent = text;
    status.textContent = `${file.name}: ${file.size} bytes, read locally.`;
  } catch {
    if (current === revision) status.textContent = 'The file could not be read.';
  }
}
input.addEventListener('change', () => show(input.files[0]));
drop.addEventListener('dragover', event => event.preventDefault());
drop.addEventListener('drop', event => {
  event.preventDefault();
  show(event.dataTransfer.files[0]);
});
</script>
```

Expected result: selecting or dropping a text file shows its contents as text. The normal input is a keyboard-accessible alternative to dropping. A revision counter prevents an earlier slow read from replacing a newer selection. The client-side extension check serves this demo's usability; it is not an upload security boundary. The [File API guide](https://developer.mozilla.org/en-US/docs/Web/API/File_API/Using_files_from_web_applications) covers selection and local file access.

## Add upload deliberately

For a small upload, create `FormData`, append the `File`, and pass it as a fetch body. Let the browser set the multipart content type and boundary. Include the application's authentication and CSRF protections, check the HTTP status, and handle cancellation with an `AbortController`. Aborting the request does not guarantee that the server discarded bytes already received.

For large files, consider chunking, resumable protocols, signed storage URLs, checksums, and a server-owned upload session. Keep keys narrowly scoped and short-lived. Show determinate progress only when measured; standard fetch upload progress is not a portable assumption. An XMLHttpRequest-based implementation can use its upload progress events where appropriate.

Image previews often use `URL.createObjectURL(file)`; revoke the URL when replaced or discarded. Do not revoke it before the image has consumed it. Image dimensions and decoding can consume substantial memory even when the compressed file is small. Process large files incrementally or in a worker when feasible.

## Rich-text editing and reordering

`contenteditable` creates an editing surface, not a complete editor model. Selection, composition input, paste normalization, undo history, keyboard behavior, and collaboration are difficult to implement correctly. Model a structured document and use a maintained editor framework when the scope warrants it. Sanitize imported HTML with a suitable maintained sanitizer and enforce a server-side policy before publication.

Use `input` and composition-aware behavior instead of assuming every keystroke produces one character. Do not persist raw DOM selection offsets as a durable collaborative document position. Reordering interfaces need move-up/down or equivalent keyboard controls, clear announcements, and a stable focus target. Dragging alone excludes users.

**Exercise:** turn the preview into a cancellable upload interface backed by your development API. Acceptance criteria: invalid files fail on both client and server, retry does not duplicate publication, names render as text, keyboard users can complete every action, and replacing a preview releases old resources.

**References:** [FormData](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest_API/Using_FormData_Objects), [Input events](https://developer.mozilla.org/en-US/docs/Web/API/Element/input_event), [OWASP file upload guidance](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html).
