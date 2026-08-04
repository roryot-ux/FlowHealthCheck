
# Flow Health Check

A simple tool that helps you look inside exported **Power Automate flows** so you
can review governance, architecture, and maintenance details. Power Automate
exports come as a `.zip` file (which contains the flow's JSON definition files),
so Flow Health Check lets you open that ZIP and see what's inside.

You give it a `.zip` file, and it tells you what's inside — the number of files,
all the filenames, the folder structure, and it highlights the JSON files (the
important parts of a flow export).

It's designed to be easy: there's nothing to install, no account to create, and
no server to set up. Built as a learning project with Claude Code.

---

## What it does

When you upload a ZIP file, Flow Health Check shows you:

- ✅ A success message
- The **name** of the file you uploaded
- The **number of files** inside the ZIP
- The number of **JSON files** inside the ZIP (flow definitions are JSON)
- A list of **all the filenames**, laid out as a **folder tree**
- **JSON files highlighted** so they're easy to spot — click one to view it
- **Flow Metrics** for a Power Automate export: flow name, triggers, actions,
  conditions, loops, scopes, variables initialised, and connectors used
- **Health Checks** with a Green / Amber / Red status for renamed actions, scope
  usage, nested loops, and error handling — each card **expands** to show the
  exact items behind its result

---

## How to run it

You don't need to install anything.

1. Find the file called `index.html` in this project folder.
2. **Double-click it.** It will open in your web browser (Chrome, Edge, Firefox, etc.).
3. Click the box to choose a ZIP file, or drag a `.zip` file onto it.
4. That's it — the results appear right away.

> **Tip:** If double-clicking opens the wrong program, right-click `index.html`,
> choose **Open with**, and pick your web browser.

You do **not** need an internet connection — it works completely offline.

---

## What files are in the project

| File | What it's for |
|------|----------------|
| `index.html` | The entire application. All the code (page layout, styling, and the logic that reads ZIP files) lives in this one file. |
| `README.md` | This file — the guide you're reading now. |
| `CHANGELOG.md` | A version-by-version record of how the app has evolved. |
| `samples/` | Example Power Automate export ZIPs you can upload to try the app. |

The whole app is a single HTML file (`index.html`), which is why there's nothing
to install. The other files are just documentation and test samples.

---

## Version history

### What Version 1 did

The first version could:

- Let you upload a ZIP file (by clicking or drag & drop)
- Display the **uploaded filename**
- Display the **number of files** inside the ZIP
- Show a **success message**

### What Version 2 added

Version 2 kept everything from Version 1 and added:

- A list of **all filenames** found in the ZIP
- The **folder structure**, shown as an indented tree (folders and files)
- **JSON files highlighted**, with their own icon and a `JSON` badge
- A count of how many **JSON files** were found

### What Version 3 added

Version 3 kept everything from Version 2 and added:

- **Click any JSON file** in the tree to open it in a viewer
- The JSON is shown **formatted with indentation** so it's easy to read
- **Syntax highlighting** (keys, text, numbers, true/false, and null are coloured)
- Files are opened **safely** — the contents are only displayed, never run, and
  the viewer does not analyse or change the flow (that may come later)

### What Version 4 added

Version 4 kept everything from Version 3 and added:

- A **Flow Metrics** section of summary cards at the top of the results
- It automatically finds and reads the Power Automate flow definition and shows
  objective counts: **flow name, triggers, actions, conditions, loops, scopes,
  variables initialised, and connectors used**
- A friendly note when the ZIP isn't a recognisable flow export

### What Version 5 added

Version 5 kept everything from Version 4 and added:

- A **Health Checks** section below the metrics, each shown as
  **Green (good) / Amber (warning) / Red (issue)**
- Checks: **renamed actions** (default names still in use), **scope usage**,
  **nested loops** (with nesting depth), and **error handling**
  (scope-based Try/Catch, basic failure handling, or none)
- All checks are **deterministic** — no AI, and no scoring yet

### What Version 6 added

Version 6 kept everything from Version 5 and added:

- Each Health Check card is now **expandable** (collapsed by default) with a
  ▶ / ▼ indicator — click the header to open or close it
- Expanding a card shows the **exact items** behind its result: the
  default-named actions, the detected scopes, each loop and its nesting depth,
  or the scopes/actions that provide error handling
- Green checks expand too, showing a short explanation

> A full, more technical version-by-version history lives in `CHANGELOG.md`.

---

## Your files stay private

Flow Health Check runs **entirely in your web browser, on your own computer.**

When you select a ZIP file, it is read locally on your machine to count and list
the files. **Nothing is uploaded to the internet or sent to any server.** The app
doesn't even need an internet connection to work — so your exported flows stay
private.

---

## Possible future features

Ideas that could be added later:

- **A health score** summarising the checks into a single rating
- **More health checks** (e.g. missing descriptions, hard-coded values, risky
  connectors, `Do until` / `Switch` handling)
- Show the **size** of each file inside the ZIP
- A **breakdown by file type** (how many JSON files, images, documents, etc.)
- **Search or filter** the file list
- **Warnings** for common problems (empty ZIP, missing expected files, etc.)
- **Export** the results as a report

---

*Flow Health Check is a small, local tool — simple by design.*

