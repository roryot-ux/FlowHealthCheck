# Changelog

A record of how **Flow Health Check** has evolved. Newest version first.

The whole app lives in a single file (`index.html`) with no build step, no
server, and no external libraries — everything runs locally in the browser.

---

## Version 3 — JSON Viewer

**Features added**
- Click any JSON file in the tree to open it in a modal viewer.
- JSON is pretty-printed with 2-space indentation.
- Syntax highlighting for keys, strings, numbers, booleans, and null.
- Viewer closes via the ✕ button, the backdrop, or the Esc key; JSON files are
  keyboard-focusable and open with Enter/Space.

**Technical changes**
- Parser now captures each entry's compression method, sizes, and local-header
  offset (including the ZIP64 extra field) so file *contents* can be located.
- Added `extractEntryBytes()`, which decompresses entries using the browser's
  built-in `DecompressionStream` (`deflate-raw`) — still no external libraries.
- The loaded ZIP buffer is kept in memory; the matching entry is attached to each
  leaf node so it can be read on click.
- Content is HTML-escaped before rendering, so file contents cannot inject markup.

**Known limitations**
- Requires a browser with `DecompressionStream` (current Chrome/Edge/Firefox);
  the viewer reports an error on very old browsers.
- Displays JSON only — no analysis of the flow yet.
- Invalid JSON is shown as raw text with a warning rather than parsed.

---

## Version 2 — Contents & Structure

**Features added**
- Lists all filenames found in the ZIP.
- Renders the folder structure as an indented tree (folders first, then files).
- Highlights JSON files with an icon and a `JSON` badge.
- Adds a "JSON files" count.

**Technical changes**
- Parser changed from counting entries to returning a full list of
  `{ name, isDir }` entries.
- Added tree-building from flat ZIP paths and recursive rendering, reconstructing
  folders even when the ZIP has no explicit folder entries.

**Known limitations**
- Shows filenames only — file contents cannot be opened yet.

---

## Version 1 — Upload & Count

**Features added**
- Upload a ZIP via click or drag & drop.
- Displays the uploaded filename.
- Displays the number of files inside the ZIP.
- Shows a success message.

**Technical changes**
- Single self-contained `index.html`: layout, styling, and logic in one file.
- Pure-JavaScript ZIP reader that counts files by parsing the central directory
  (with ZIP64 support); folder entries are excluded from the count.
- Everything runs in the browser — no server, no install, works offline; the ZIP
  is never uploaded anywhere.

**Known limitations**
- Only the filename and file count are shown — no contents or structure.
