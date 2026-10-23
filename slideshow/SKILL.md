---
name: slideshow
description: Turn a source document (transcript, report, guide, article, meeting notes, PDF text) into a Reveal.js HTML slide deck. Produces two files — a structured outline in markdown and a presentable HTML deck — with review checkpoints between stages. Use this skill whenever the user wants to convert written or spoken content into slides, build a presentation from existing material, summarize a video or article as a deck, or create a visual walkthrough of a document. Trigger on phrases like "make slides", "turn this into a presentation", "build a deck from this", "slideshow from the transcript", "presentation version of this report", even when the user doesn't explicitly say "Reveal.js". This skill is the default answer for any deck-generation task unless the user specifically asks for PowerPoint/.pptx output.
---

# Slideshow

Convert a source document into a Reveal.js slide deck through a four-stage pipeline with two review checkpoints. The goal is to produce slides that *earn* their existence — every slide carries one idea, every visual has a reason to exist, and the structure reflects how the source actually flows.

## Why this skill exists

Slide decks made directly from a transcript or report tend to fail in the same way: too many bullets, no visual rhythm, no sense of narrative. The pipeline here forces separation of concerns — first decide what the deck says, then decide what it shows, then render. Review gates exist because errors at the structure stage compound downstream. A wrong outline produces wrong visuals produces a wrong deck. Catching structure issues early is orders of magnitude cheaper than fixing them after rendering.

## Inputs accepted

Any text source the user can hand over:
- Pasted transcript or article text
- Uploaded `.txt`, `.md`, `.pdf`, or `.docx` file
- URL to a transcript file
- A mix of the above when building a deck from multiple sources

If the source is a YouTube video or other media requiring fetching, ask the user to paste the transcript or use a separate ingestion skill — this skill starts from text.

## Outputs produced

Exactly two files, both saved to the user's workspace or a location they specify:

1. **`<deck-name>-outline.md`** — the structured intermediate format. Contains deck metadata, sections with summaries and key points, visual descriptions, and speaker notes. This is the source of truth; the HTML deck is rendered *from* it. Users can edit the outline and re-render.

2. **`<deck-name>.html`** — the Reveal.js deck as a single self-contained HTML file. Loads Reveal.js from CDN, uses the clean-light theme bundled in `assets/deck-template.html`. Opens by double-clicking in any browser. Exports to PDF via `?print-pdf` URL parameter in Chrome. Hosts on any static site.

Name the files based on the source topic, kebab-cased. Example: source is a transcript about database indexing → `database-indexing-outline.md` and `database-indexing.html`.

## Pipeline

The pipeline has four stages with review gates after stages 2 and 3. Do not skip gates unless the user explicitly says "skip review" or "go straight to the deck."

### Stage 1 — Ingest

Read the source document. If the source is long (>5000 words), read it in full before proceeding — the structure stage depends on grasping the whole arc. Don't summarize yet; just absorb.

Note the type of source. Transcripts, reports, and tutorials each suggest different structures:
- **Transcript** (talk, interview, video): narrative flow, speaker voice preserved in notes
- **Report or guide**: section-driven, denser content, often expository
- **Article or essay**: argument-driven, one central thesis with supporting moves
- **Meeting notes or messy text**: extract the coherent threads, drop the noise

This shapes stage 2.

### Stage 2 — Structure (Outline)

Produce the outline as `<deck-name>-outline.md` using the format in `assets/outline-template.md`. The outline includes:

- **Deck metadata**: title, subtitle, source, audience, estimated length
- **Section list**: ordered sections with one-line summaries
- **Per-section details**: section title, 2-4 key points, suggested slide count

Slide-count heuristics for adaptive length:
- Opening (title + context) ≈ 2-3 slides
- Each major section ≈ 3-6 slides depending on density
- Transitions between sections ≈ 0-1 slide
- Closing (summary + next steps) ≈ 2-3 slides

For a 30-minute talk with 4-5 major sections, expect 20-30 total slides.

**Content density per slide** follows the "one idea per slide" rule:
- Title + 3-5 bullets, OR
- Title + one focused paragraph, OR
- Title + one quote/stat/image with minimal surrounding text

Slides with 6+ bullets are a signal to split or reduce. Slides with sub-bullets are a signal to split.

**REVIEW GATE 1 — after outline.** Save `<deck-name>-outline.md`, then ask the user (using AskUserQuestion if available, otherwise conversational prompt):

> I've drafted the outline — [N] sections across [M] slides. Want to review the structure before I write visual descriptions, or should I proceed?

Options: "Looks good, proceed" / "Let me review first" / "Let's restructure" / "Change scope (shorter/longer)".

If the user wants changes, revise the outline in place and re-prompt. Do not proceed to stage 3 without explicit approval.

### Stage 3 — Visual descriptions

For each slide that warrants a visual, write a visual description in the outline. Add these directly to `<deck-name>-outline.md` under each slide.

**Default visual strategy: placeholders + descriptions.** Do not generate or embed real images unless the user has asked for that explicitly or provided them. Instead, for each visual slot, write a clear, specific description the user can use to find or generate an image later. The HTML deck will render a placeholder box with the description text visible.

Visual description format: one line, concrete, specific enough that the user could hand it to a stock photo search or an image generator and get something useful.

- Weak: "A picture of a server"
- Strong: "An isolated server rack in a dim datacenter, shot low-angle with cool blue ambient light, suggesting isolation and scale"

**Not every slide needs a visual.** Title slides, quote slides, and text-dense summary slides often read better without images. Use visual slots deliberately — typically 30-60% of slides.

**Slides that strongly benefit from visuals:**
- Section openers (sets the emotional tone for what follows)
- Concept slides (a picture clarifies an abstract idea)
- Example/case-study slides (grounds the abstract in the concrete)
- Process or flow slides (consider a mermaid diagram instead — see below)

**Mermaid diagrams** are a first-class alternative to image placeholders for processes, flows, org structures, timelines, or relationships. Use them when the relationship between elements is the content. Write the mermaid code directly in the outline; the template supports rendering it inline.

**REVIEW GATE 2 — after visual descriptions.** Ask the user:

> I've added visual descriptions to [N] slides. Want to review them before I render the deck, or should I proceed to the HTML?

Options: "Looks good, render it" / "Let me review first" / "Change visual style" / "Add/remove visuals".

Skip this gate only if the user previously said "skip review" or the deck has fewer than ~8 slides (at that size, the round trip isn't worth it — render and let them review the final deck).

### Stage 4 — Render

Read `assets/deck-template.html`. Populate it section by section from the outline. Save to `<deck-name>.html`.

Rendering rules:

- Each slide is a `<section data-markdown>` block containing markdown content. This keeps the source readable and lets the user hand-edit slides later.
- Nested sections (vertical slides) are used for drill-downs within a section. Create them by nesting `<section>` inside `<section>`. Use sparingly — one level of nesting max.
- Speaker notes go inside `<aside class="notes">`. For transcript sources, the speaker notes should preserve the source's voice; don't over-summarize them.
- Visual placeholders render as a styled div with the description text. The CSS for this is in the template.
- Mermaid diagrams render via the mermaid plugin (loaded in the template).

After rendering, verify:
- The file opens in a browser without console errors
- All section tags are balanced
- No markdown syntax is leaking visibly into rendered slides (common failure: forgetting to escape `---` inside a slide, which breaks the deck)

## Customization

Users may ask for variations. Common ones:

- **Different theme**: swap the CSS link in the `<head>` or edit the custom CSS block. The template ships with clean-light; `black`, `white`, `league`, `beige`, `night`, `serif`, `simple`, `solarized`, `moon`, `dracula`, `blood` are available from CDN.
- **Offline-portable deck**: inline the Reveal.js library and CSS into the HTML. Produces a larger file but works without network.
- **PDF export**: append `?print-pdf` to the URL in Chrome, then Print → Save as PDF. Document this in the README output.
- **Change aspect ratio**: set `width`/`height` in the `Reveal.initialize()` call (default 960×700).

## Output file sharing

After rendering, share both files with `computer://` links:

```
Outline: [View the outline](computer://path/to/deck-name-outline.md)
Deck: [View the deck](computer://path/to/deck-name.html)
```

Briefly note PDF export instructions so the user knows how to hand the deck off.

## Failure modes to avoid

- **Over-chunking from transcripts**: a 30-minute talk should not become 60 slides because every topic shift became a new section. Merge minor shifts. Aim for the 3-5 major beats.
- **Under-editing speaker voice**: transcripts are messy. Clean them for bullets; preserve them for speaker notes. Don't paste raw transcript as slide text.
- **Visual spam**: resist putting a stock-photo placeholder on every slide. Empty space is fine. Visuals with no reason to exist make the deck worse, not better.
- **Bullet overload**: if a slide has 6+ bullets, it's two slides or it's a paragraph. Fix it.
- **Skipping review gates silently**: the user should know when a gate is skipped and why.
