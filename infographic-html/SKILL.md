---
name: infographic-html
description: >
  Create polished, self-contained HTML infographics in a dark technical design system. Use this skill whenever the user wants to visualize a process, workflow, methodology, phases, steps, or structured reference content as an infographic — even if they just say "make an infographic", "visualize this", "turn this into a diagram", or "create a visual for this doc". Also use it when iterating on an existing infographic (layout changes, color swaps, content updates, section additions/removals). The output is always a single downloadable HTML file.
---

# HTML Infographic Skill

Produce beautiful, self-contained HTML infographics using the established dark technical design system. All output is a single `.html` file with embedded CSS — no external dependencies except Google Fonts.

---

## Design System

### Fonts
Always import both from Google Fonts:
- **Syne** (400, 600, 700, 800) — body, headers, labels
- **JetBrains Mono** (400, 600, 700) — pills, badges, code, mono elements

```html
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&family=Syne:wght@400;600;700;800&display=swap" rel="stylesheet">
```

### Color Palette (CSS Variables)
Always define these in `:root`. Swap accent values per-project but keep the structure:

```css
:root {
  --bg: #0d0f14;          /* page background */
  --surface: #13161e;     /* card/panel background */
  --border: #1f2433;      /* dividers, outlines */
  --text-primary: #e8eaf0;
  --text-muted: #7a8099;
  --text-dim: #4a5068;
  --converge: #a78bfa;    /* purple — used for convergence/summary elements */
}
```

**Standard accent palette** (assign to sections based on role/sequence):
| Color | Hex | Typical use |
|---|---|---|
| Blue | `#38bdf8` | Phase 1 / primary section |
| Green | `#4fffb0` | Phase 2A / new/positive |
| Orange | `#f97316` | Phase 2B / existing/legacy |
| Purple | `#a78bfa` | Convergence / summary |
| Pink | `#f472b6` | Optional 4th section |
| Yellow | `#fbbf24` | Optional 5th section |

### Structural Patterns

**Card container:**
```css
.infographic {
  width: 100%;
  max-width: 1200px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 12px;
  overflow: hidden;
}
```

**Body layout options:**
- Two-column: `grid-template-columns: 1fr 2fr` (left panel + right main)
- Three-column: `grid-template-columns: 1fr 1fr 1fr`
- Single column with stacked sections

**Section with accent background glow:**
```css
background: linear-gradient(160deg, rgba(R,G,B,0.03) 0%, transparent 60%);
```
Use the section's accent color RGB values for the subtle tint.

### Component Patterns

**Phase / section label (mono, small-caps):**
```css
font-family: 'JetBrains Mono', monospace;
font-size: 9px;
font-weight: 700;
letter-spacing: 0.25em;
text-transform: uppercase;
color: var(--accent);
margin-bottom: 6px;
```

**Section title:**
```css
font-size: 22px;
font-weight: 800;
letter-spacing: -0.03em;
line-height: 1.1;
color: var(--accent);
```

**Pill / badge (mono):**
```css
font-family: 'JetBrains Mono', monospace;
font-size: 9px;
font-weight: 700;
letter-spacing: 0.2em;
text-transform: uppercase;
padding: 3px 10px;
border-radius: 3px;
color: var(--accent);
background: rgba(R,G,B,0.1);
border: 1px solid rgba(R,G,B,0.3);
```

**Numbered step circle:**
```css
width: 22px; height: 22px;
border-radius: 50%;
background: rgba(R,G,B,0.15);
border: 1px solid rgba(R,G,B,0.4);
color: var(--accent);
font-family: 'JetBrains Mono', monospace;
font-size: 10px; font-weight: 700;
display: flex; align-items: center; justify-content: center;
```

**Icon box (for bullet lists):**
```css
width: 24px; height: 24px;
border-radius: 5px;
background: rgba(R,G,B,0.1);
border: 1px solid rgba(R,G,B,0.25);
display: flex; align-items: center; justify-content: center;
flex-shrink: 0;
```
Use 11×11px inline SVG icons inside, `stroke="currentColor"`, `color: var(--accent)`.

**Goal / callout box (dashed border):**
```css
padding: 16px;
border: 1px dashed rgba(R,G,B,0.25);
border-radius: 7px;
background: rgba(R,G,B,0.04);
font-size: 11.5px;
color: var(--accent);
line-height: 1.6;
font-style: italic;
```

**Horizontal steps row:**
```css
.steps { display: flex; gap: 0; align-items: stretch; }
.step  { flex: 1; }
/* Divider between steps: */
.step:not(:last-child)::after {
  content: '';
  position: absolute;
  top: 16px; right: -1px;
  width: 1px; height: 32px;
  background: var(--border);
}
```

**Convergence footer:**
```css
border-top: 1px solid var(--border);
padding: 16px 32px;
display: flex; align-items: center; gap: 14px;
background: rgba(167,139,250,0.04);
```
Include artifact/output tags as purple mono pills.

---

## Layout Decision Guide

| Content shape | Recommended layout |
|---|---|
| One intro phase + two parallel paths | Left 1/3 panel + right 2/3 split top/bottom |
| Linear sequence of phases | Horizontal steps row or vertical stack |
| Comparison (A vs B) | Two equal columns |
| Hub + spokes | Central box with radial items (use absolute positioning or CSS grid) |
| Multi-phase funnel | Stacked sections narrowing toward bottom |

---

## File Output Rules

- Single `.html` file, all CSS embedded in `<style>` tag
- No JavaScript required (pure CSS layout)
- `body` uses flexbox to center `.infographic` on the page
- `max-width: 1200px` on the card; page `padding: 32px`
- Save to `/mnt/user-data/outputs/<slug>-infographic.html`
- Present with `present_files` after creation

---

## Workflow

1. **Read the source content** — project file, pasted text, or conversation context
2. **Identify structure** — how many sections, are there parallel paths, is there a convergence point?
3. **Choose layout** — pick from the Layout Decision Guide above
4. **Assign accent colors** — one color per major section, in sequence from the standard palette
5. **Build the HTML** — apply all design system patterns; write clean, commented HTML
6. **Output the file** — save and present

### Iteration
When the user requests changes (layout, colors, content, section removal/addition):
- Read the current file with `view` first
- Make targeted `str_replace` edits rather than rewriting the whole file
- Re-present after changes

---

## Quality Checklist

Before presenting:
- [ ] Both fonts imported from Google Fonts
- [ ] All colors via CSS variables (no hardcoded hex in layout rules)
- [ ] Accent RGB values match the CSS variable hex in `rgba()` calls
- [ ] Section dividers use `var(--border)`
- [ ] Subtle gradient background glow on each section
- [ ] Mono font on all pills, badges, step numbers, code snippets
- [ ] Icon SVGs are 11×11px, `stroke="currentColor"`, `fill="none"`, `stroke-width="2.5"`
- [ ] File is self-contained (no external CSS or JS files)
