# Skills

A list of the AI skills I have been developing and using.

Each skill is a folder containing a `SKILL.md` (plus any templates or specs it
needs). Drop a folder into your agent's skills directory to use it.

## Skills

### [`brainstorm`](brainstorm/SKILL.md)

Runs an idea-generation session: exploring possibilities, generating options,
and pressure-testing half-formed plans. Gives ideas in batches of five to eight
distinct options with a stated recommendation, but asks only one question per
turn. Researches the web when outside input would sharpen the ideas. Strictly
read-only — it will not create, edit, or delete any file until you explicitly
say to. Companion to `conversation`, which governs how the questions get asked.

### [`conversation`](conversation/SKILL.md)

Governs how the agent asks for information in any back-and-forth — scoping,
requirements gathering, troubleshooting, planning. Three rules: no multiple
choice or option-picker widgets, one question per turn, and every question
arrives with a recommendation and the reasoning behind it. Applies whenever a
reply would otherwise contain a clarifying question.

### [`git-main`](git-main/SKILL.md)

Safely returns to the repo's default branch and pulls the latest changes.
Auto-detects `main` vs `master` from the `origin` remote rather than assuming,
warns and lists files first if there are uncommitted changes, and reports the
resulting branch, commit, and how many commits came in.

### [`infographic-html`](infographic-html/SKILL.md)

Creates polished, self-contained HTML infographics in a dark technical design
system — for visualizing processes, workflows, methodologies, or phased
reference content. Defines the full design system: Syne + JetBrains Mono, a CSS
variable palette, and component patterns for pills, step circles, callout boxes,
and convergence footers. Includes a layout decision guide and a pre-delivery
quality checklist. Output is always a single `.html` file with embedded CSS.

### [`okf-html-library`](okf-html-library/SKILL.md)

Maintains a personal, browsable knowledge library in the Open Knowledge Format
(HTML profile), where each concept is a themed HTML file you read in a browser.
Handles adding and updating concepts, regenerating indexes, and organizing
around topic hubs — main topics get a hub page and a folder, and the root index
stays a clean map rather than a pile. Styling lives only in a shared
`theme.css`; all links are relative so the library works over `file://`.
Ships with `SPEC.md` and `templates/` (concept, index, theme).

## [`Development/`](Development/README.md) — the AI development process

A pipeline of nine skills that carries a vague app idea through interviews,
design handoffs, planning, and a PRD into an unattended plan → track → execute
implementation loop. See the [Development README](Development/README.md) for the
full pipeline, each skill, and the principles behind it.
