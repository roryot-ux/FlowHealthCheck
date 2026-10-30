# Changelog

A record of how **Flow Health Check** has evolved. Newest version first.

The whole app lives in a single file (`index.html`) with no build step, no
server, and no external libraries — everything runs locally in the browser.

---

## Version 8 — Multi-Flow Solutions

**Features added**
- Power Platform **solution ZIPs containing multiple flows** are now supported:
  every distinct flow in the ZIP is detected, not just the first.
- When more than one flow is found, a **"N flows detected"** label and a
  **dropdown** appear at the top of Flow Metrics; selecting a flow immediately
  updates the metrics and health checks for that flow.
- Flows are listed alphabetically; flows sharing a display name are labelled
  with their source file path so they can be told apart.
- Single-flow ZIPs keep the existing experience — no selector is shown.
- Tested with a solution containing 17 flows.

**Technical changes**
- `locateFlowDefinition()` became `locateFlowDefinitions()`, returning an array
  of `{ definition, name, label, source, connectionReferences }`. Both export
  shapes (package `properties.definition` and solution `workflows/*.json` with
  stringified `clientdata`) are still resolved by the unchanged
  `resolveDefinition()`.
- The likely flow files (`definition.json`, `Microsoft.Flow/flows/`,
  `workflows/`) are scanned first; remaining JSON is only scanned if none of
  those held a flow, so unrelated JSON isn't mistaken for an extra flow.
- New `collectFlows()` de-duplicates flows by their serialised definition, so
  the same flow stored in more than one file appears once.
- `showFlowMetrics()` now handles discovery and the selector; rendering moved to
  a new `renderFlow(flow)`, re-run on the dropdown's `change` event. The
  metrics and health-check analysis itself is unchanged.
- Small `.flow-picker` CSS block reusing the existing palette. Still no external
  libraries; everything stays local in the browser.

**Known limitations**
- No connection-reference or environment-variable analysis yet.
- De-duplication matches identical definitions only; two copies of a flow that
  differ in any way are listed separately.
- There's no summary across all flows — metrics and health checks are shown one
  flow at a time.

---

## Version 7 — XML Preview

**Features added**
- XML files are now **clickable** in the contents tree, just like JSON files —
  highlighted with an `XML` badge (blue, to distinguish them from JSON's
  yellow) and openable with Enter/Space as well as a click.
- Clicking one opens it in the **same modal viewer** used for JSON, with the
  same ✕ / backdrop / Esc close behaviour.
- XML is shown with its **own indentation preserved** exactly as it appears in
  the file; minified single-line XML is given one indent per nesting level so
  it's readable.
- Basic XML **syntax highlighting**: element names, attribute names, attribute
  values, and comments.

**Technical changes**
- The tree's `isJson` boolean became `fileKind()`, returning `"json"`, `"xml"`
  or `null`; the icon, badge, CSS class and click/keyboard handlers in
  `buildList()` are all driven from that one value, so both file types share a
  single code path.
- `openJson()` became `openFile(entry, kind)`. The modal shell — title,
  loading state, decompression, warning note and error handling — is shared;
  only the formatting step branches, to `renderJson()` or `renderXml()`, each
  returning `{ html, warning? }`. The JSON path is unchanged.
- `formatXml()` returns files that already contain line breaks untouched, so
  significant whitespace and CDATA survive; only single-line input is
  re-indented, by splitting on `>\s*<` and tracking depth.
- `highlightXml()` escapes first, then makes a single pass with one alternating
  regex (comments | CDATA | tags) so markup inside a comment or CDATA block
  isn't coloured as a tag. Within a tag, attributes are coloured before the
  anchored element name, so the pattern can't match the class names of the
  `<span>`s just inserted.
- Extracted the shared `escapeHtml()` helper (now used by both highlighters);
  `syntaxHighlight()` renamed to `highlightJson()`. New `.tok-tag`,
  `.tok-attr`, `.tok-val` and `.tok-comment` colours reuse the existing
  palette. Still no external libraries.

**Known limitations**
- Malformed XML is shown on a best-effort basis with **no warning**, unlike the
  JSON viewer which flags files it can't parse.
- A literal `>` inside an attribute value (valid XML, but rare) ends the tag
  early, so that tag is coloured incorrectly. Display only — no content is lost.
- Re-indenting minified XML is best-effort rather than a full parse, so
  whitespace inside a comment or CDATA block in a single-line file may shift.
- The stats still count JSON files only — there's no "XML files" count.

---

## Version 6 — Expandable Health Checks

**Features added**
- Each Health Check card is now **expandable**, collapsed by default, with a
  ▶ / ▼ indicator; clicking (or Enter/Space on) the header toggles it.
- Expanding a card reveals the exact items behind its status:
  1. **Renamed actions** — the action names still using default names.
  2. **Scope usage** — the names of the detected scopes.
  3. **Nested loops** — each loop with its nesting depth.
  4. **Error handling** — the scopes/actions providing handling, or an
     explanation of why the check has no handling to show.
- Green checks are expandable too (they show a short explanation).

**Technical changes**
- `analyzeHealth()` / `walkHealth()` now collect item lists instead of just
  counts: `unrenamedNames`, `scopeNames`, `loops` (`{name, depth}`), and
  `errorHandlers` (`{name, kind}`).
- Each check now returns `{ status, summary, details[] }`; `status` is
  `good` / `warning` / `issue` (mapped to the existing colour classes).
- `buildHealthItem()` rebuilt as a collapsible card: clickable header +
  details list, animated with a CSS `max-height` transition, keyboard
  accessible via `aria-expanded`. Detail lines use `textContent` (no markup
  injection). No external libraries.

**Known limitations**
- Still no scoring (each check stands alone).
- Detail lines list raw action keys (underscored) as they appear in the
  definition JSON.
- Carries forward the V5 detection limits (default-name coverage,
  `Failed`-based error-handling detection, `Foreach`-only loops).

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
