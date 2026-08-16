---
name: okf-html-library
description: Maintain a personal, browsable HTML knowledge library in the Open Knowledge Format (HTML profile). Use whenever the user wants to file away, save, add, capture, or note down knowledge into their library, or asks to update, reorganize, look something up in, or browse their library. Triggers include "file this", "add this to my library", "save a note on", "put this in my knowledge base", "update the entry on", "what do I have about".
---

# OKF-HTML Library

This skill maintains a personal knowledge library where each concept is
a themed HTML file the user reads in a browser. The full format is
specified in `SPEC.md` next to this file — read it if you need detail.
This document is the operating manual: how to add, update, and organize
concepts on the user's request.

## Library location

The library is a directory containing a root `theme.css` and one or more
`index.html` files. Before editing, determine the library root:

1. If the user names a folder, use it.
2. Otherwise look for a directory containing `theme.css` + `index.html`
   in the user's workspace (a `library/` folder alongside this skill is
   the default).
3. If none exists yet, offer to create one: copy `templates/theme.css`
   to `<root>/theme.css`, create a root `index.html` from
   `templates/index.html`, and confirm the location with the user.

## Core rules (from SPEC.md)

- Each concept is one `.html` file. Concept ID = path minus `.html`.
- Every concept has an `okf-meta` JSON block in `<head>` with a required
  `type` field (see §4.1 of the spec). Keep `<title>` in sync with the
  metadata `title`.
- Every file links the root `theme.css` with a **relative** path, and
  all cross-links between concepts are **relative** (`../`, `./`). Never
  use absolute-from-root (`/...`) paths — they break on `file://`.
  Compute `../` depth from where the file sits.
- Structure the body with semantic HTML: headings, tables, lists,
  `<pre><code>`, `<figure><img>`, inline SVG/`<canvas>` for graphs.
- Never hand-write styling into a concept — styling lives only in
  `theme.css`. If the user wants a different look, edit `theme.css`, not
  the concepts.

## Topic hubs — the standard organization

Organize the library around **main topics, each with a hub page**. This
keeps the root index a clean, scannable map while letting each topic
grow unlimited depth inside its own folder.

- A main topic gets its own subdirectory containing a **hub page**
  (e.g. `World Building/world-building.html`, type `Playbook` or
  similar). The hub is the topic's front door: a short overview, the
  distilled advice or takeaways, and links to every subtopic page in
  the folder.
- The root `index.html` lists **main topics only**, each entry linking
  the topic's hub page. Subtopic pages never appear on the root index —
  they are reachable only through their hub. If the root index listed
  every page, it would stop being a map and become a pile.
- Each summarized **source** (a video, article, talk, book chapter)
  gets its own page in the topic folder, holding the detailed notes —
  timestamps, quotes, examples — with the original URL as its
  `okf-meta` `resource` and linked in the body. The hub keeps only a
  condensed takeaway and references the source page. This way detail
  is never lost, but the hub stays readable.
- Breadcrumbs mirror the hierarchy: root index → topic hub → subtopic.
  When the topic folder has no `index.html` of its own, the hub page is
  the breadcrumb's middle link.
- When a root-level concept accumulates subtopics, **promote it**:
  create the topic folder, move the page in as its hub, move or create
  the subtopic pages beside it, recompute `theme.css` hrefs, breadcrumb
  and cross-link depths, and update the root index entry to point at
  the hub's new path.

Small libraries can start with everything at the root; introduce a hub
the moment a topic gains its second page.

## Adding a concept

1. Decide where it belongs. A subtopic or source summary goes in its
   main topic's folder and gets linked from that topic's hub page (see
   "Topic hubs" above); standalone concepts can group in subdirectories
   (e.g. `topics/`, `people/`, `recipes/`). Ask only if genuinely
   ambiguous; otherwise pick a sensible location.
2. Choose a short, lowercase, hyphenated filename → the concept ID.
3. Copy `templates/concept.html` and fill in the placeholders:
   - `{{TYPE}}` — a short, self-explanatory type (`Note`, `Reference`,
     `Person`, `Recipe`, `Playbook`, …). Reuse types the library
     already uses when they fit; check existing files first.
   - `{{TITLE}}`, `{{DESCRIPTION}}` — display name and one-line summary.
   - `{{TAGS}}` — JSON string list, e.g. `"coffee", "brewing"` (empty is
     fine).
   - `{{RESOURCE}}` — a quoted URI, or `null` if abstract.
   - `{{TIMESTAMP}}` — current time, ISO 8601 (get it via a shell
     `date -u +%Y-%m-%dT%H:%M:%SZ` if unsure).
   - `{{THEME_HREF}}` — relative path to `theme.css` for this file's
     depth (`theme.css`, `../theme.css`, …).
   - `{{BREADCRUMB}}` — the trail back up to the root index, rendered at
     the top of the page. Link every ancestor index with a **relative**
     path (same depth rule as `{{THEME_HREF}}`), separate them with
     `<span class="sep">/</span>`, and leave the current page as plain
     text rather than a link. A root-level concept gets a single
     ancestor link; a concept in `topics/` gets two.
   - `{{TAG_CHIPS}}` — one `<span class="okf-tag">tag</span>` per tag,
     or leave empty. These render in the chip bar at the **bottom** of
     the page, alongside the type chip.
   - `{{BODY}}` — the actual content as semantic HTML.
4. Regenerate the affected `index.html` (see below) — unless the
   concept is a subtopic of a hub, in which case update the hub page's
   links instead of the root index.
5. Append an entry to `log.html` if the library keeps one.

## Updating a concept

Edit the existing file in place. When you make a meaningful change,
update the `timestamp` in the `okf-meta` block. If the title or
description changed, refresh the matching `index.html` entry and the
`<title>` tag. If the file moved, recompute both the `theme.css` href
and the breadcrumb links for its new depth.

## Maintaining index.html

Whenever you add, rename, move, or remove a concept, regenerate the
`index.html` in that directory (and the root index if a top-level entry
changed). Build it from `templates/index.html`: group entries under
`<h2>` headings, one `<li>` per concept or subdirectory, each linking
the target (relative path) and showing its metadata `description`. Pull
descriptions from each target's `okf-meta` block so the index stays
accurate. Subdirectory entries link the folder (`subdir/`) — except
topic folders with a hub page, which are listed as the topic name
linking the hub page itself, with the hub's description. The root
index lists main topics and standalone root concepts only; subtopic
pages are indexed by their hub, not here.

Only the root `index.html` carries an `okf-meta` block and the
`okf_html_version` declaration; subdirectory indexes may omit it.

## Looking things up

When the user asks what the library contains or to find something, start
from the root `index.html`, follow links, and read the `okf-meta` blocks
and bodies. Tags and `type` values are your filters — scan `okf-meta`
blocks across files to answer "what do I have about X".

## After editing

Tell the user what changed and point them to the file to open in their
browser (present the concept or the root `index.html`). Keep the summary
short — they can read the page themselves.

## Rich content notes

- Images: save alongside the concept (or in an `assets/` folder) and
  reference with a relative `<img src>`.
- Graphs/motion: prefer inline SVG or a small inline `<canvas>`/JS
  snippet within the concept when it's specific to that page; if several
  concepts share charting logic, factor it into a shared script at the
  root and link it relatively. Keep purely visual styling in `theme.css`.
- Keep each concept a self-contained, valid HTML document so it opens
  correctly on its own.
