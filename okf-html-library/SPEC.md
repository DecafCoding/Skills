# Open Knowledge Format — HTML Profile (OKF-HTML)

**Version 0.1 — Draft**

OKF-HTML is a profile of the Open Knowledge Format designed for a
**personal, browsable knowledge library**. It keeps OKF's core ideas —
self-describing concepts, permissive consumption, a plain directory tree
— but stores each concept as an HTML file so the library can be read
directly in a browser, with images, themes, graphs, and motion.

The design goal is a library that is *pleasant to read* and *maintained
by an agent*. You tell the agent what to file; the agent writes and
updates the HTML. You open `index.html` and browse.

---

## 1. What changed from base OKF

Base OKF stores concepts as markdown (`.md`) with YAML frontmatter and
optimizes for `cat`-in-a-terminal readability and clean diffs. OKF-HTML
makes a different trade for a single-user, browser-first library:

| Concern            | Base OKF                     | OKF-HTML                                  |
|--------------------|------------------------------|-------------------------------------------|
| Concept file       | `<name>.md`                  | `<name>.html`                             |
| Metadata           | YAML frontmatter             | JSON in a `<script>` block in `<head>`    |
| Index              | `index.md`                   | `index.html`                              |
| Log                | `log.md`                     | `log.html` (optional)                     |
| Links              | markdown links               | `<a href>` — native browser navigation    |
| Styling            | none                         | shared `theme.css` linked by every file   |
| Rich content       | limited                      | images, SVG/canvas graphs, CSS, JS        |

Everything else — the permissive consumption model, `type` as the only
required field, tolerance of unknown fields and broken links — carries
over unchanged.

---

## 2. Terminology

- **Library** — The whole collection. Equivalent to a base-OKF "bundle".
  The unit you browse and back up.
- **Concept** — One unit of knowledge, stored as one `.html` file.
- **Concept ID** — The concept's path within the library with the
  `.html` suffix removed. `topics/coffee.html` has ID `topics/coffee`.
- **Metadata block** — The JSON `<script>` in the `<head>` carrying the
  concept's structured fields (the OKF-HTML equivalent of frontmatter).
- **Body** — The rendered content inside `<main>`.
- **Theme** — The shared `theme.css` at the library root that styles
  every concept.

---

## 3. Library structure

A library is a directory tree of `.html` files plus one `theme.css` at
the root.

```
my-library/
├── theme.css                # REQUIRED. Shared styling, linked by every file.
├── index.html               # Home page / table of contents.
├── log.html                 # Optional. Chronological update history.
├── <concept>.html           # A concept at the library root.
└── <subdirectory>/          # Subdirectories group related concepts.
    ├── index.html
    ├── <concept>.html
    └── <subdirectory>/
        └── …
```

### 3.1 Reserved filenames

| Filename      | Purpose                          |
|---------------|----------------------------------|
| `index.html`  | Directory listing. See §6.       |
| `log.html`    | Update history. See §7.          |
| `theme.css`   | Shared stylesheet (root only).   |

All other `.html` files are concept documents.

### 3.2 Links and paths — use relative paths

Because the library is meant to be opened **directly from disk**
(`file://`), all internal links and the `theme.css` link MUST be
**relative**, not absolute-from-root. A leading `/` resolves to the
filesystem root under `file://`, not the library root, and would break.

- A concept at the root links the theme as `href="theme.css"`.
- A concept one level deep links it as `href="../theme.css"`.
- Cross-links between concepts are ordinary relative paths, e.g.
  `href="../topics/coffee.html"` or `href="./sibling.html"`.

The agent computes the correct number of `../` from the file's depth.

---

## 4. Concept documents

Every concept is a UTF-8 HTML file with two required parts: a **metadata
block** in the `<head>` and a **body** inside `<main>`.

### 4.1 The metadata block

Structured fields live in a single JSON object inside a
`<script type="application/json" id="okf-meta">` element in the `<head>`.
This is the OKF-HTML replacement for YAML frontmatter: one predictable,
trivially parseable place for metadata.

```html
<script type="application/json" id="okf-meta">
{
  "type": "Reference",
  "title": "Pour-over ratios",
  "description": "Coffee-to-water ratios for common pour-over methods.",
  "tags": ["coffee", "brewing"],
  "resource": null,
  "timestamp": "2026-07-01T12:00:00Z"
}
</script>
```

**Required:**

- `type` — A short string identifying the kind of concept (e.g.
  `Reference`, `Note`, `Playbook`, `Person`, `Recipe`). Not registered
  centrally; consumers MUST tolerate unknown types.

**Recommended:**

- `title` — Display name. Also used in `<title>` and the page heading.
- `description` — One sentence; used in `index.html` entries.
- `tags` — List of short strings for cross-cutting grouping.
- `resource` — Canonical URI of an underlying asset, or `null`/omitted
  for abstract concepts.
- `timestamp` — ISO 8601 datetime of last meaningful change.

Producers MAY add any other keys. Consumers MUST NOT reject a document
for unknown keys or missing optional fields.

### 4.2 Head boilerplate

Every concept `<head>` SHOULD contain, in addition to the metadata block:

```html
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>…the title…</title>
<link rel="stylesheet" href="…relative path to theme.css…">
```

Keeping `<title>` in sync with the metadata `title` is recommended so
browser tabs and history read well.

### 4.3 Body

The body is the content inside `<main>`. Structure it with semantic HTML
— `<h1>`…`<h3>`, `<ul>`, `<table>`, `<pre><code>`, `<figure><img>`,
inline SVG, `<canvas>` — since structure aids both reading and agent
retrieval. Start the body with an `<h1>` matching the title.

Conventional sections (use when applicable), expressed as headings:

| Heading      | Purpose                                             |
|--------------|-----------------------------------------------------|
| `Schema`     | Structured description of an asset's fields.        |
| `Examples`   | Concrete usage examples.                            |
| `Citations`  | External sources backing claims. See §8.            |

### 4.4 Example concept

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Pour-over ratios</title>
  <link rel="stylesheet" href="../theme.css">
  <script type="application/json" id="okf-meta">
  {
    "type": "Reference",
    "title": "Pour-over ratios",
    "description": "Coffee-to-water ratios for common pour-over methods.",
    "tags": ["coffee", "brewing"],
    "timestamp": "2026-07-01T12:00:00Z"
  }
  </script>
</head>
<body>
  <main>
    <h1>Pour-over ratios</h1>
    <p>Ratios are expressed as grams of coffee to grams of water.</p>
    <table>
      <thead><tr><th>Method</th><th>Ratio</th><th>Notes</th></tr></thead>
      <tbody>
        <tr><td>V60</td><td>1:16</td><td>Medium grind.</td></tr>
        <tr><td>Chemex</td><td>1:17</td><td>Coarser grind.</td></tr>
      </tbody>
    </table>
    <p>See also <a href="./grinders.html">grinders</a>.</p>
  </main>
</body>
</html>
```

---

## 5. Cross-linking

Concepts link to each other with ordinary `<a href>` using relative
paths (§3.2). A link asserts a relationship; the *kind* of relationship
is conveyed by the surrounding prose, not the link. Consumers MUST
tolerate broken links — a link to a not-yet-written concept is valid,
not an error.

---

## 6. Index files

An `index.html` MAY appear in any directory, including the library root.
It enumerates that directory's contents so a reader can see what exists
before opening individual concepts, and serves as the browsable home
page.

An index lists each concept and subdirectory as a link with its
description (pulled from the target's metadata `description`), grouped
under headings as makes sense. The root `index.html` is the front door
of the library.

The agent regenerates the relevant `index.html` whenever it adds,
renames, or removes a concept.

---

## 7. Log files (optional)

A `log.html` MAY appear at any level to record changes to that scope:
date-grouped entries, newest first, each a short line naming what
happened and linking the affected concept. Dates use ISO 8601
`YYYY-MM-DD`.

---

## 8. Citations

When a concept makes claims from external sources, list them under a
`Citations` heading at the end of the body as a numbered list of links.
Citations MAY be external URLs or relative links into a `references/`
subdirectory of mirrored material.

---

## 9. Conformance

A library conforms to OKF-HTML v0.1 if:

1. A `theme.css` exists at the library root.
2. Every non-reserved `.html` file contains an `okf-meta` metadata block
   that parses as JSON.
3. Every metadata block has a non-empty `type`.
4. Internal links and the theme link are relative (§3.2).

Consumers MUST NOT reject a library for: missing optional fields,
unknown `type` values, unknown metadata keys, broken cross-links, or
missing `index.html` files. This permissive model is intentional — the
library stays usable as it grows and is continuously edited by an agent.

---

## 10. Versioning

This document specifies OKF-HTML **0.1**. Minor bumps add
backward-compatible options; major bumps may change required fields or
reserved filenames. A library MAY declare its target version by adding
`"okf_html_version": "0.1"` to the root `index.html`'s `okf-meta` block.
