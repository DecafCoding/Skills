# slideshow

Turn a source document into a Reveal.js slide deck.

## What it does

Takes any written or spoken source — transcripts, reports, guides, articles, meeting notes, PDFs — and produces:

1. A structured outline in markdown (`<name>-outline.md`), editable and re-runnable
2. A presentable Reveal.js HTML deck (`<name>.html`), portable and self-contained

## How to use it

Just ask. Any of these phrasings will invoke the skill:

- "Turn this transcript into slides"
- "Make a deck from this report"
- "Build a presentation from this article"
- "Slideshow version of this guide"

Hand over the source — paste it, upload a file, or point at a text file.

## What happens

Four stages with two review gates:

1. **Ingest** — Claude reads the source.
2. **Structure** — Claude produces the outline. *You review before proceeding.*
3. **Visuals** — Claude writes visual descriptions for each slide that warrants one. *You review before rendering.*
4. **Render** — Claude produces the HTML deck.

If the deck is small (<8 slides) or you say "skip review," the second gate is elided.

## Files produced

```
<deck-name>-outline.md    # structured content, editable
<deck-name>.html          # presentable deck, double-click to open
```

Both land in your current workspace unless you specify otherwise.

## Viewing and presenting

**Open the deck:** double-click the `.html` file. Any modern browser works. No internet required for basic viewing (the library loads from CDN by default; ask for an "offline" deck if you need everything inlined).

**Present:**

- `F` — fullscreen
- `S` — speaker view (shows notes + upcoming slide on a second window)
- `Arrow keys` — navigate
- `Esc` or `O` — overview of all slides
- `?` — show keyboard shortcuts

**Export to PDF:** open the deck in Chrome, append `?print-pdf` to the URL, then Print → Save as PDF. Reveal.js formats it as proper slides, one per page.

**Host on the web:** the HTML file is self-contained. Drop it on GitHub Pages, Netlify, Vercel, S3, or any static host. Works anywhere that serves a single file.

## Editing after the fact

The outline markdown is the source of truth. To change the deck:

1. Edit `<deck-name>-outline.md`
2. Ask Claude to "re-render the deck from the outline"

You can also hand-edit the HTML directly for small tweaks, but structural changes belong in the outline.

## Customization

**Theme:** the default is a clean minimal light look (white bg, terracotta accent, Inter + Charter typography). Ask for variations: "dark", "editorial serif", "bold playful", or provide your own CSS.

**Aspect ratio:** default is 1100×720. Ask for 16:9 (1920×1080), 4:3 (1024×768), or a custom size.

**Offline deck:** ask for an "offline" or "portable" deck and Claude will inline the Reveal.js library and CSS into the HTML. Larger file, no network needed.

## Customizing the skill itself

The skill files live in `skills/slideshow/`:

- `SKILL.md` — the instructions Claude follows. Edit this to change the pipeline, review gates, slide-density rules, or content conventions.
- `assets/deck-template.html` — the starting point for every rendered deck. Edit to change the default visual style, layouts, or plugins.
- `assets/outline-template.md` — the shape of the intermediate format. Edit to change what metadata or structure the outline captures.

After editing, the next invocation of the skill will pick up the changes automatically.

## Known limits

- Source must be text. This skill does not fetch YouTube transcripts or scrape web pages on its own — paste or upload the text.
- Images are placeholders by default. If you want real images, either provide them or ask Claude to generate descriptions you can hand to an image generator.
- The Reveal.js library is loaded from a CDN by default. For deployments where CDN access is restricted, ask for an offline deck.
