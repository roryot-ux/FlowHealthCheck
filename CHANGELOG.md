# Changelog

A record of how **Flow Health Check** has evolved. Newest version first.

The whole app lives in a single file (`index.html`) with no build step, no
server, and no external libraries — everything runs locally in the browser.

---

## Version 5 — Health Checks

**Features added**
- New "Health Checks" section below Flow Metrics, using a Green / Amber / Red
  status per check.
- Checks implemented (deterministic, no AI):
  1. **Renamed actions** — flags actions still using default names
     (Compose, Condition, Scope, Apply to each, etc., with optional trailing
     number) and reports how many.
  2. **Scope usage** — reports whether scope actions are present and how many.
  3. **Nested loops** — detects `Apply to each` nested inside another and
     reports the nesting depth; warns when nested.
  4. **Error handling** — reports scope-based Try/Catch (Configure Run After),
     basic failure handling (Run After = Failed), or none.
- Added `samples/sample-flow-with-issues.zip` to exercise the warning paths.

**Technical changes**
- `analyzeHealth()` / `walkHealth()` do a single recursive pass over the action
  tree, carrying loop-nesting depth and inspecting each action's `runAfter`.
- Default names matched via a regex built from a base-name list, comparing the
  action key with underscores converted to spaces.
- `renderHealthChecks()` builds status rows via DOM APIs (no innerHTML for
  data), coloured by status class.

**Known limitations**
- No scoring yet (each check stands alone).
- Default-name detection covers common control/data actions only, not every
  connector's default operation name.
- Error-handling detection keys on `runAfter` containing "Failed"; unusual
  patterns may not be recognised.
- "Nested loops" considers `Foreach` only (matching the metrics definition).

---

## Version 4 — Flow Metrics

**Features added**
- New "Flow Metrics" section of summary cards at the top of the results.
- Automatically locates and reads the Power Automate flow definition JSON.
- Reports objective counts: flow name, triggers, actions, conditions, loops
  (Apply to each / Foreach), scope actions, variables initialised, and
  connectors used.
- Shows a friendly note when the ZIP isn't a recognisable flow export.
- Added `samples/sample-flow-export.zip` for testing.

**Technical changes**
- `locateFlowDefinition()` scans JSON entries and resolves the definition from
  both export shapes: package (`properties.definition`) and Dataverse/solution
  (`workflows/*.json` with a stringified `clientdata`).
- `analyzeFlow()` / `walkActions()` recurse the action tree, descending into
  Scope, If (`else`), Foreach/Until, and Switch (`cases`/`default`).
- Connectors are collected from action/trigger `inputs.host` and from
  `connectionReferences`, then normalised (strip `shared_` prefix and guid-style
  suffixes) and de-duplicated via a Set.
- Reuses the V3 `extractEntryBytes()` decompressor to read the JSON.

**Known limitations**
- Objective counts only — no governance analysis yet.
- "Conditions" counts `If` and "loops" counts `Foreach` only (per spec); `Switch`
  and `Until` are not counted as conditions/loops.
- "Actions" is the total of all action nodes, including control actions
  (scopes, conditions, loops).
- Connector detection is best-effort and may miscount unusual/custom connector
  id formats.

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
