---
name: dev-claud-md
description: Generate a project-specific CLAUDE.md guidance file from the project's settled architecture (docs/architecture.html), its PRD (docs/prd.html) and, when it exists, the repository itself. Prefers docs/architecture.html over the PRD and over its own .NET/Vertical Slice house default for stack, pattern, layout and deployment, so an agreed architecture is never silently overwritten by a default. Use whenever the user wants to create, regenerate, refresh, or fix the CLAUDE.md / project guidance / agent instructions file for a repo — including phrasings like "write the CLAUDE.md", "set up Claude Code for this project", "the agent keeps forgetting our conventions", or "give this repo its house rules". Also use it right after a PRD is created and before the first phase is executed in the PRD to dev-plan-phase to dev-create-progress to dev-execute pipeline, so every downstream agent run inherits the same conventions. Stack-adaptive — emits a .NET/C# profile by default and equivalent profiles for other stacks.
---

# Create CLAUDE.md: Project Guidance For Coding Agents

## Overview

Generate the `CLAUDE.md` that Claude Code reads on every session in a given repository. The file captures **the decisions and gotchas a new contributor could not guess from the code alone** — not generic advice.

The primary input is the project's PRD (default `docs/prd.html`, produced by `/dev-create-prd`). When a repository already exists, read it too and prefer observed reality over PRD intent.

This skill **never writes application code**. It writes exactly one file: `CLAUDE.md`.

## Arguments

- **First argument (optional):** Path to the PRD. Default `docs/prd.html`. May also be a folder of design docs, or another single document.
- **Second argument (optional):** Output path. Default `CLAUDE.md` at the repository root.

If the default PRD path does not exist and no argument was given, look for `docs/prd.md`, `docs/prd.html`, `README.md`, and any `docs/*.html` plan documents before asking the user where the requirements live.

## Step 0: Gather Ground Truth

### Read The PRD

Read the PRD in full. Extract:

| What To Extract | Where It Lands In CLAUDE.md |
|---|---|
| Product summary, audience, platform | Project Overview |
| Core flow or pipeline | Project Overview (one-line Input to Output) |
| Technology Stack section | Stack profile selection, Build & Test |
| Core Architecture & Patterns | Architecture, Key Patterns |
| Security & Configuration | Configuration, Authentication |
| Directory structure | Repository Location & Layout |
| Testing approach | Testing |

### Read The Architecture Doc First, When It Exists

Check for `docs/architecture.html` before the PRD. When `dev-architecture` has run, that file — not the PRD — is the authority on stack, storage, architecture pattern, module layout, boundaries, state and failure, non-functional targets and deployment. The PRD summarizes it; a summary is not the source.

Read every `<h3 class="decision">` and take its `data-decision` slug, `data-status` and `data-reversible`. Then:

| What To Extract | Where It Lands In CLAUDE.md |
|---|---|
| Stack | Stack profile selection, Build & Test |
| Architecture pattern, Module layout | Architecture, Key Patterns |
| Boundaries, State and failure | Key Patterns |
| Deployment, Non-functional | Configuration, Authentication |

**This overrides the house default in Step 1.** The .NET / Vertical Slice default exists for projects that never ran `dev-architecture`. When the file exists and names something else, the file wins — silently defaulting over an agreed decision is the one failure this check prevents. Say in your closing summary which source you used.

A decision with `data-status="open"` is not settled. Do not write it into CLAUDE.md as though it were; leave the section out and name it in the closing summary.

If `docs/architecture.html` does not exist, carry on with the PRD as before.

### Scan The Repository (When One Exists)

If the working directory is a repo, verify the PRD's claims against it. Look for, as applicable to the stack: solution/project files, `package.json`, `pyproject.toml`/`requirements.txt`, `go.mod`, `Cargo.toml`, `Makefile`, `launchSettings.json`, `appsettings.json`, `.env.example`, `docker-compose.yml`, the test directory, and any existing `CLAUDE.md`.

**Observed reality beats PRD intent.** If the PRD says PostgreSQL and the connection string says CockroachDB, write what the repo does and mention the discrepancy in your closing summary.

If no repo exists yet, build the file from the PRD's planned structure and say so in the closing summary — the paths are the plan, not a scan.

### Handle An Existing CLAUDE.md

If the output file already exists, read it first. Preserve any hand-written content that is still accurate — especially hard-won gotchas the PRD does not know about. Update in place; never silently drop a section. If something in the existing file now conflicts with the repo, flag it and ask before removing it.

## Step 1: Pick The Stack Profile

Choose in this order of authority: `docs/architecture.html` first, then the PRD's Technology Stack section, then the repo. Where they disagree, prefer the higher authority and name the disagreement in your closing summary — except against the repo, where observed reality still wins for anything already built.

- **.NET / C#** — the default and richest profile. Use it whenever `docs/architecture.html` or the PRD names .NET, ASP.NET Core, Blazor, WPF, MAUI, or EF Core — and as the fallback when neither source states a stack at all.
- **Python**, **Node / TypeScript**, **Go**, **Rust**, **other** — emit the same *sections*, with the stack's own commands and conventions substituted (see "Stack Profiles" below).

If the project is genuinely polyglot (for example, a .NET API plus a React front end), emit one Build & Test subsection per component rather than averaging them into something true of neither.

## Step 2: Resolve Every Placeholder

The template below uses `{curly brace}` placeholders. **No placeholder may survive into the written file.** For each one:

1. Fill it from the PRD or the repo scan.
2. If it cannot be determined and the section does not apply, **delete the whole section**. A short accurate file beats a long hedged one.
3. If it cannot be determined but the section clearly does apply, ask the user.

### How To Ask

Ask **one question at a time**, conversationally, in prose. Attach your recommended answer and a one-line reason for it, then let the user type a free-text reply.

**Never** present multiple-choice lists, lettered or numbered options to pick from, or tappable/one-click answer buttons — not in this skill, and not in the file it produces.

Good: *"I could not find a test project. I'd recommend xUnit with plain `Assert` and EF Core Sqlite for data tests — it matches the rest of the stack and avoids the InMemory provider's value-converter blind spot. What are you using?"*

Bad: any variant that hands the user a menu instead of a conversation.

## Step 3: Write CLAUDE.md

Write the file at the output path using the structure below. Keep it **short and concrete** — prefer *"use the `DbContext` directly, no repository pattern"* over *"write clean code"*. Target roughly 100 to 200 lines; delete every section that does not apply.

### Mandatory Sections (Never Omit, Never Soften)

These three sections appear in **every** generated CLAUDE.md regardless of stack, PRD content, or repo state. Emit them essentially verbatim.

````markdown
## Commit Authorship

- All commits are authored **solely by the user**. Do **not** add Claude, Claude Code, or any
  AI tool as an author, and do **not** append a `Co-Authored-By` trailer for one.
- **No AI attribution anywhere.** Not in commit messages, PR or issue descriptions, code
  comments, XML/doc comments, changelogs, release notes, or documentation. Do not add
  "Generated with…", "🤖", "AI-assisted", or any equivalent marker.
- Write commit messages in the repository owner's voice: what changed and why.

## UI Conventions

- **Title Case For All UI Text** — labels, button text, section headings, page titles, menu
  items, table column headers, tab names, dialog titles, and toast/notification titles.
  Example: "Classic Movies From The 1990's", **not** "Classic movies from the 1990s".
- Title Case applies to headings in generated documentation and reports as well.
- Body copy, help text, validation messages, and log messages stay in sentence case.

## Working With Me (Interaction Rules)

- **Never** present multiple-choice questions, lettered or numbered pick-lists, or
  one-click/tappable answer options.
- Ask questions **conversationally, one at a time**, and wait for the answer before asking
  the next one.
- **Every question carries a recommendation** — your suggested answer plus a one-line reason.
- Always leave room for a typed, free-text reply; never constrain the answer to a fixed set.
````

When the project has no user interface at all, keep the UI Conventions section but scope it to CLI output, generated documents, and log/report headings.

### Full Template

Fill this out, then delete what does not apply.

````markdown
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

{One or two sentences: what this application is, who it is for, and the platform.}

{If there is a pipeline or core flow, one line end-to-end: **Input → StageA → StageB → Output.**}

See `README.md` for setup/usage and `docs/prd.html` for the authoritative spec.

## Repository Location & Layout

The repo lives at `{path}`. {Solution/manifest file}. Main project(s): `{name}`. Tests live
under `{tests path}`.

## Build & Test

```bash
{build command}
{test command}
{run command}
{format/lint command, or "none configured"}
```

Run URLs: `{urls, if a web app}`.

### {Migrations — delete if no database}

```bash
{migration add command}
{migration apply command}
```

{State whether migrations apply automatically at startup or must be run manually. If more than
one database context exists, call that out — every command must name the context, and it is a
common foot-gun.}

## Architecture

{Overall style in one line, plus the big "no"s — e.g. "Vertical Slice; no MediatR, no
repository pattern — use the DbContext directly".}

{Folder-to-namespace/module mapping and key locations, one bullet per top-level folder.}

### Key Patterns

{Only the non-obvious ones. Candidates: state machines and status transitions; what is caught
versus what fails the whole operation; how background work is started and its scope/lifetime
concerns; data-driven registries and plug-in points; serialized columns and their query
limitations; external API clients and where secrets live.}

## {Dependency Injection / Module Wiring — delete if not applicable}

{Explicit lifetimes where they matter, and the hazards. For .NET, always include the
captive-dependency warning: never inject a scoped service such as `DbContext` into a singleton;
open a fresh scope with `IServiceScopeFactory` instead.}

## Configuration

{Where settings live, how they bind to typed options, and the secrets rule: non-secret defaults
ship in source, secrets are blank in source and supplied via the stack's secret mechanism.
Include the exact command to set a secret and the env-var equivalent.}

## Authentication

{Scheme, what is gated, and any flows that must NOT be modified. Delete if none.}

## Conventions

- {Async/concurrency rule for all I/O.}
- {Where interfaces are allowed — typically only at the boundary you actually mock.}
- {Logging rule — a logger per class; never write directly to stdout.}
- {Config rule — typed options only, no scattered environment lookups.}
- {A brief doc comment atop every file/class.}
- **No secrets in source or committed config.**

## Testing

- {Framework, assertion style, mocking library, data strategy.}
- {Known limitations of the test data strategy.}
- {How network/DB-touching logic is isolated for testing.}
- Name tests for behavior: `MethodName_Condition_ExpectedResult`.

## UI Conventions

{Mandatory block — see above.}

## Working With Me (Interaction Rules)

{Mandatory block — see above.}

## Commit Authorship

{Mandatory block — see above.}
````

## Stack Profiles

Every profile emits the same sections; only the specifics change.

### .NET / C# (Default)

- **Build & Test:** `dotnet build`, `dotnet test`, `dotnet run --project {name}`, `dotnet format`. Note the solution file (`.sln` or `.slnx`) explicitly.
- **Migrations:** EF Core `dotnet ef migrations add` / `dotnet ef database update`. **If more than one `DbContext` exists, every command must pass `--context` — call this out as a foot-gun.** Note whether `Database.Migrate()` runs at startup.
- **Architecture default:** Vertical Slice — organize by feature under `Features/{FeatureName}/`, keeping endpoint/page, handler, models, and validation together. Extract to `Shared/` only for genuine cross-cutting concerns, never prematurely. **This is a default, not an override.** If `docs/architecture.html` settled a different pattern or layout, use that one and say so in the closing summary.
- **DI:** thread-safe clients singleton; API clients as typed `HttpClient` via `AddHttpClient<>`; handlers and `DbContext` scoped; pure helpers `static`. Always include the captive-dependency warning.
- **Config:** `appsettings.json` bound to a typed options class; `dotnet user-secrets set` for secrets; `SECTION__KEY` env-var equivalents; `ConnectionStrings:DefaultConnection`.
- **UI:** Blazor apps use **Bootstrap** for styling and layout — grid, components, and utilities — rather than another CSS framework or hand-rolled CSS.
- **Conventions:** async for all I/O with explicit return types; `ILogger<T>` per class, never `Console.WriteLine`; interfaces only at the network boundary.
- **Testing:** xUnit/NUnit/MSTest with the repo's assertion and mocking libraries; EF Core InMemory vs Sqlite vs Testcontainers. If InMemory is used, warn that it ignores value converters — assert the entity graph, not the serialized column shape.

### Python

- **Build & Test:** the project's runner (`uv`, `poetry`, `pip` + venv), `pytest`, `ruff`/`black`, `mypy` if configured.
- **Migrations:** Alembic (`alembic revision --autogenerate`, `alembic upgrade head`) or the ORM's equivalent.
- **Architecture:** package layout and the import-direction rule (which layers may import which).
- **Wiring:** how dependencies are provided — FastAPI `Depends`, a container, or plain construction — and any request-scoped session rules.
- **Config:** `pydantic-settings` or equivalent, `.env` and `.env.example`, secrets never committed.
- **Conventions:** async boundaries, type hints required, module-level `logging.getLogger(__name__)`, never `print`.

### Node / TypeScript

- **Build & Test:** the package manager actually in use (`npm`/`pnpm`/`yarn` — check the lockfile), plus `build`, `test`, `lint`, `typecheck` scripts from `package.json`.
- **Migrations:** Prisma, Drizzle, or TypeORM commands as applicable.
- **Architecture:** module/route structure, server/client boundary for full-stack frameworks, and `strict` TypeScript expectations.
- **Config:** typed env parsing (zod/envalid), `.env.example`, and which vars are client-exposed.
- **Conventions:** no `any` without justification, structured logging over `console.log`, error-handling boundary.

### Go / Rust / Other

Emit the same sections using the stack's idioms: build/test/lint commands, module layout, dependency wiring, config and secrets handling, logging rule, and test naming. If you are unsure of a convention, ask rather than inventing one.

## Quality Checks

Before reporting completion, verify:

- No `{curly brace}` placeholder survives anywhere in the file.
- Every command listed actually exists in the repo (correct project names, correct solution file, correct package manager per the lockfile).
- The three mandatory sections — Commit Authorship, UI Conventions, Working With Me — are present and unsoftened.
- No section is generic filler. If a bullet would be true of any project, cut it.
- Sections that do not apply were deleted, not left as empty headings.
- The file is roughly 100 to 200 lines.
- Any section carried over from a pre-existing CLAUDE.md is still accurate.

## Output Confirmation

After writing the file:

1. Confirm the path written.
2. List the sections included and the sections deleted as not applicable.
3. Report any discrepancies found between `docs/architecture.html`, the PRD, and the repo — and which one you wrote.
4. State plainly which source the stack and pattern came from: `docs/architecture.html`, the PRD, or the house default. If the default was used because no architecture doc exists, say so and mention `dev-architecture`.
5. Note assumptions made where the sources were silent, and any decision left out because it was `data-status="open"`.
6. Suggest next steps — typically `/dev-plan-phase` for the next phase, so the phase work runs under these conventions.

## Notes

- Written **before** the first `/dev-execute` run wherever possible; every agent session afterward inherits the conventions.
- Re-run it after a phase that meaningfully changes the architecture, adds a second database context, or introduces a new component to the stack.
- Authority order for technical decisions: `docs/architecture.html` beats the PRD, and the repo beats both for anything already built. Architecture is the source of decision, the PRD is the source of intent, the repo is the source of truth. When they disagree, write the repo and say so.
- Re-run it after `dev-architecture` lands an amendment — an amended decision that never reaches `CLAUDE.md` means every later agent session works from the superseded convention.
- This skill writes only `CLAUDE.md`. It never writes application code, tests, or migrations.
