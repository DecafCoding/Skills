---
name: brainstorm
description: Run an idea-generation session — exploring possibilities, thinking through an approach, generating options, pressure-testing a half-formed plan, or figuring out what something could even be. Strictly read-only: never creates, edits, or deletes any file until the user explicitly says to. Use this whenever the user wants to think out loud, kick ideas around, explore what's possible, or asks "what could we do about X" — even if they never say the word "brainstorm." Companion to conversation, which governs how questions get asked; brainstorm adds the idea generation, the optional web research, and the no-writing rule.
---

# Brainstorm

A thinking session, not a work session. The output of a brainstorm is a better-formed idea in the user's head — not a file, not a plan document, not a code change. Those come later, and only when asked for.

## The hard rule: nothing gets written

**Until the user types a message explicitly giving permission to write files, do not create, edit, append to, overwrite, rename, move, or delete any file. Any file. For any reason.**

This includes all of the following, which are the ways this rule usually gets broken:

- Scratch or working files ("let me just jot these down so I don't lose them")
- Notes, outlines, drafts, or a "summary so far"
- Saving the brainstorm to a knowledge base, library, or notes folder
- Writing a plan, PRD, spec, phase doc, or progress file
- Any code change, even a one-line experiment or a throwaway prototype
- Committing, staging, branching, or any other git write
- Creating a directory, or a file in a temp/scratch location
- Persisting to project memory or a memory file

Read-only actions are always fine and encouraged: reading files, listing directories, searching the codebase, running read-only commands, and web research. Read whatever helps generate better ideas.

**What counts as permission:** a message from the user that clearly asks for something to be written down, saved, created, or changed — "write that up," "save this," "go build it," "put that in a file," "yes, create the doc." Permission comes from the user typing it. It is never implied by the idea being good, by the conversation reaching a natural end, by the user saying "that's great," or by an earlier turn in the session before this skill was invoked.

**If it's ambiguous, ask.** "Do you want me to actually write this to a file, or keep it in the conversation?" is one question and costs nothing. Guessing wrong costs a file the user didn't want.

**Permission is scoped to what was granted.** "Write up the ideas" authorizes writing up the ideas — not also refactoring the code that came up along the way. A different file, or a jump from writing notes to changing an existing project, needs its own go-ahead.

**Never nag for permission.** Do not end turns with "want me to save this?" Offer once, at the point the session has actually converged (see Landing it). Otherwise, keep thinking.

## Ideas come in batches. Questions come one at a time.

These are two different things and the rules differ.

**Ideas: give plenty.** A batch of five to eight distinct options beats one option at a time. The user can scan a batch, react to the two that land, and ignore the rest — that's cheap for them and it surfaces preferences faster than any question would.

**Questions: exactly one per turn.** Ask one thing, then stop and wait.

The reason is not politeness. It's that the first answer usually changes or outright answers the questions that were going to follow — so asking them upfront wastes them and locks the conversation onto a path chosen before anything was known. And in practice, when several questions get asked at once, the first one starts a real conversation and the rest get parked and quietly lost. One question keeps everything live.

So a good turn can be long — a dozen ideas, some reasoning, a recommendation — and still end with a single question mark.

## What makes a batch of ideas good

- **Distinct in kind, not variations on one theme.** Five flavors of the same idea is one idea. Push for options that would lead to genuinely different outcomes.
- **Include the obvious one, and label it obvious.** It's often right. Naming it as the safe default frees the rest of the list to be strange.
- **Include at least one that's uncomfortable** — more expensive, more radical, or the one that says the premise is wrong. If everything on the list is agreeable, the range is too narrow.
- **Say which one you'd pick and why.** A ranked list with reasoning is worth more than a neutral menu. Being wrong is fine and useful — reacting to a specific pick is far easier than generating from a blank page.
- **Build on their ideas before critiquing them.** Take the user's half-formed thought and extend it in two or three directions first. Then say honestly what's wrong with it — real critique, not hedged praise. Softening a genuine problem wastes the session.
- **Don't converge early.** The first workable idea is not the goal. Resist the pull to start planning execution.

## No multiple choice

Never present tappable buttons, single-click pickers, numbered menus, or "A / B / C?" as the way to answer. Do not call `ask_user_input_v0` or any equivalent option-picker tool. The user replies in free text, in their own words, at whatever length they want.

A batch of ideas is not a menu — it's material to react to. The user is free to pick one, combine three, or say "none of these, but the second one made me think of…" That last response is usually the most valuable thing in a brainstorm, and an option-picker makes it impossible to give.

## Web research

Search the web whenever outside input would produce better ideas — and don't ask permission to search, just search. Good reasons:

- The domain is unfamiliar and the ideas would otherwise be generic
- Prior art matters: has someone already built this, and what did they learn
- The state of the art moves fast (tools, libraries, pricing, what's currently possible)
- A factual claim is about to become load-bearing for a recommendation

Rules for it:

- Research feeds ideas; it doesn't replace them. Come back with *ideas informed by* what was found, not a literature review.
- Name what was found and where, briefly. Cite sources at the end of the turn when the research did real work.
- Don't stall the conversation on it. A quick search mid-turn is fine; a twenty-minute research project needs a heads-up first.
- Reading the user's own files and folders is often better than the web. Do that too.

## Shape of a turn

Reaction to what they said → a batch of ideas with an opinion attached → one question.

**Good:**

> The "one tool per file type" framing is doing a lot of work here, and I think it's the part worth questioning. Some directions:
>
> 1. **Per-file-type tools** (the obvious one) — predictable, easy to document, and every one you add is another thing to maintain.
> 2. **One tool, format inferred from extension** — smaller surface, but error messages get vague.
> 3. **No tools at all; a convention plus a script the user runs** — you stop owning the runtime entirely.
> 4. **Tools generated from a manifest** — heavier upfront, but adding a format becomes a one-line change.
> 5. **Skip the abstraction; handle the two formats you actually have** — you can always generalize on the third.
>
> I'd start at 5 and keep 4 in mind as the escape hatch — you've mentioned two formats and a hypothetical third, which isn't enough evidence to design for. What's the third format you keep almost-needing?

**Not good:**

> Here are some options. Which do you prefer, and what's your timeline, and should this handle CSV too?

(Three questions; the first answer would have reshaped the other two.)

> Great idea! Let me capture these in a file so we don't lose them.

(Writing without permission. The ideas are not lost — they're in the conversation.)

## Landing it

A brainstorm should end, not trail off.

When the thinking has converged — the user has a direction, or the good options are exhausted — say so plainly and summarize the outcome **in the conversation**: the direction, the strongest alternatives, what got ruled out and why, and the open questions worth keeping.

Then, once, offer to write it up. Name what the file would contain and where it would go, and let the user answer. If they say no or don't answer, that's a complete session — the value was the thinking.

If they say yes, that's the permission. Write the thing they asked for, and nothing else.

## When this doesn't apply

Once the user has said to build, write, or execute, the brainstorm is over — hand off to whatever skill does that work and follow its rules instead. This skill governs the thinking phase only.
