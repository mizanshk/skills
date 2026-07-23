---
name: dev-handoff-notes
description: >-
  Write or update dev handoff notes — a designer's repository of everything a
  developer needs to build the real product from a high-fidelity prototype:
  interactions, states, accessibility, logic and rules, UX intent, data,
  edge cases, and what's mocked vs. real — written in developer-native language
  so nothing that lived in your head (or only became real in the prototype) is
  lost in the handoff. The doc is also ideal to hand to an AI coding assistant
  as build context: prototype plus this doc is what an agent needs to build the
  real thing. Use whenever someone wants to document how a prototype works for
  engineering, capture build-critical detail or edge cases before a design→dev
  handoff, or create/update a DEV-HANDOFF-NOTES.md. Works for any prototype, any
  stack (React, single-file HTML, Framer, SwiftUI, v0…), and any or no design
  system. Trigger even on just "write up how this works for the devs", "handoff
  notes", "spec this for engineering", "the handoff", or "capture the details so
  they don't get lost".
---

# Dev Handoff Notes

## What this is

A **dev handoff note** is a designer's repository of everything a developer needs to build the real product from a high-fidelity prototype — the interactions, states, accessibility, logic, UX intent, data, and edge cases that the prototype shows or implies but doesn't spell out. The developer builds from two things together: the **prototype** (what it should look like and how it behaves) and these **notes** (the details, logic, and intent behind it). You write it when you hand a prototype to engineering, so the knowledge in your head doesn't evaporate in the handoff.

It serves two readers:

- **A developer**, who builds the real product with the prototype open beside them — it speaks their language (real values, component names, behavior, rules), so they aren't reverse-engineering your intent.
- **The developer's AI coding assistant.** The prototype plus this doc is exactly the context an AI needs to build the real product; a dev can drop both straight into their coding agent.

The bar: **capture enough, precisely enough, that someone who wasn't in your head — human or AI — can build the real product from the prototype without guessing.**

## It's a spec, not a changelog

The handoff is the **opposite of a design log**. A design log is chronological decision-history — *why* things were done, what was tried and rejected, why a value is what it is. The handoff is a snapshot of *what is* — *how* to build it, right now. That distinction governs everything.

Because it's a spec, you **describe the present and keep it true**. When the prototype changes, you rewrite the stale parts — you don't append. A handoff that carries history, contradictions, or stale claims is worse than none: a developer will build the wrong thing and trust it. If you catch yourself writing "we changed X to Y" or "previously this did…", that belongs in a log — state only the current behavior here.

## What to capture

Completeness is the whole point — the details you *don't* write down are the ones that get lost. For each surface, cover whichever apply:

- **Interactions & behavior** — what happens, in what order, on what trigger.
- **Every state** — default, hover, active/selected, focus, empty, loading/skeleton, error, partial, disabled, read-only. Design frames usually show the happy path; the other states are where builds drift. Enumerate them.
- **Logic & rules** — what governs what: visibility conditions, counts, sort/filter order, validation, derived values. The "this shows only when that is true."
- **Data & state model** — the shape it expects, where state lives, what persists and how, the key entities and their relationships. Use the real field/key/entity names, not paraphrases.
- **Mocked vs. real** — what's **fake** (stubs, seed data, localStorage, canned responses, in-memory state, non-functional controls) — always paired with **what the real thing would be** (which API, endpoint, service, or auth plugs in here). This is the single most valuable section: it draws the line between where the prototype ends and production begins.
- **Accessibility** — roles, keyboard paths, focus management, ARIA. If it follows a known pattern (a WAI-ARIA tree, a focus-trapped modal), name the pattern so it can be looked up.
- **Responsive** — what changes at which breakpoints, and what is explicitly *not* handled.
- **Copy that matters** — user-visible strings that are deliberate/spec (not placeholder), plus any copy rules to preserve.
- **Edge cases & gotchas** — smallest/largest data, overflow, truncation, races, and any dead-end you hit while building (so the dev doesn't re-hit it).
- **Prototype-only bits to strip** — dev toggles, mode switches, seed/reset utilities, debug output, hardcoded users, mock auth. Anything that must **not** ship. Flag it explicitly, even if it seems obvious — these are the things that quietly ship by accident.
- **The why** — the intent behind a choice, where a dev would otherwise guess wrong. This is the one thing only you know; code doesn't carry it. Keep it to intent that constrains the build, not decision-history.

## What makes it good

- **Developer-native language.** Name the real thing — the function, component, selector, route, or state that implements the behavior — so a dev (or their AI) can find it. Anchor to the prototype's actual code, whatever the stack.
- **Exact values, not vibes.** `#4645BB`, `py-[7px]`, `min-height: 96px`, `300ms cubic-bezier(…)`. "An appropriate hover state" is unbuildable; a value is. A value lost is work lost.
- **Flag every fake, with the fix.** Prototypes lie — stubs that just toast, in-memory state, placeholder names. Each is a landmine for a dev who assumes it's real. Call it out and say how to wire it.
- **Map to the design system, if there is one.** List each UI element and the component it should reuse, so engineering reuses instead of rebuilding. No design system? Skip it entirely — don't invent one.
- **Stay current and honest.** Keep a "Last updated" line; when the doc trails the prototype, add a **Known lag** note saying exactly what's stale rather than silently misdescribing it. A doc the dev can't trust is worse than none.

## Structure

```
# Dev Handoff Notes — <Prototype>

> [Preamble] What the prototype is (stack + one-liner), the goal, the
> load-bearing architectural facts, and a **Last updated:** line (+ any
> **Known lag** flag).

## Contents            ← TOC linking every section
## Overview & architecture   ← file/route layout; how state + rendering work
## <Feature section>   ← one per surface. Repeat.
...
## Mocked vs. real           ← each fake → what plugs in for production
## Component inventory       ← element → design-system component (omit if no DS)
## Running / building        ← how to run it locally
## Strip before shipping     ← prototype-only toggles, seeds, mock auth
## Cross-cutting invariants  ← numbered "don't break these" rules that hold everywhere
```

**A feature section** opens with a **`File:`** line naming the entry point (the file / component / function / selector that implements it), then dense bullets covering the dimensions above.

**Cross-cutting invariants** is the highest-leverage section — the rules that hold everywhere and must not break (e.g. "all actions are stubs — wire them"; "one row component across every list — keep them identical"). It's how a dev avoids quietly breaking a load-bearing assumption.

## Voice

Terse, present-tense (it describes how the prototype *behaves today*), dense with specifics, no marketing.

**Good:**
> **Select-all** (`#saCbx`, `role="checkbox"`): tri-state from the filtered set — `aria-checked` `false` / `mixed` / `true`. Clicking `mixed` clears all (Gmail behavior). Selects the *whole filtered set*, not just the loaded page.

**Not:**
> There's a select-all checkbox at the top that selects everything. *(No selector, no states, no rule — unbuildable.)*

Two failure modes to avoid, both from the wrong document living inside this one:

- Reads as **"we decided to…"** → that's a design log. State what *is*, not why you got there.
- Reads as **"handles the various states"** → which states, doing what? Name each one and what it does.

## Feeding it to an AI builder

A dev can hand this file to their AI coding assistant as build context, so keep it **self-contained and unambiguous**: real names/paths, exact values, states and rules spelled out, fakes flagged. Anything left vague, the model will invent — the same precision that helps a human helps the AI. This is the use case worth optimizing for: "prototype + this doc → an agent builds the real thing" is likely why you're writing it.

## Writing a new note

The prototype is the source of truth — read it before you write. Even when the prototype was built in this very conversation, don't write from memory: open the entry points (the files, components, routes, or selectors each feature's `File:` line will name) and pull the real values, the state shape, which controls are wired and which are stubbed. Then ask the designer only for what the code can't reveal — the intent behind a choice, what's deliberately unhandled, what production thing each mock stands in for. A note written from memory instead of the code is exactly how you get the vague, unanchored spec this skill exists to prevent.

Put a new doc at the repo root as `DEV-HANDOFF-NOTES.md` (or beside the prototype if there's no repo root), so the search order below finds it next time.

## Updating an existing note

Most handoff work is updates, and the discipline is **reconcile, never append** — because this is a spec, not a record. The running prototype is the source of truth; when the doc and the code disagree, the code wins and the doc gets corrected.

Before writing a fresh doc, look for an existing one — `DEV-HANDOFF-NOTES.md` at the repo root, then a `docs/` folder, then the working directory. When you find one, update it in place rather than writing a second file beside it — two handoffs for one prototype is the exact contradiction this doc exists to prevent. If two plausible docs exist (say a broad `HANDOFF.md` and a `DEV-HANDOFF-NOTES.md`), confirm which is the developer handoff before touching it.

When you update, be **surgical**: touch only the sections your change affects, match the existing structure and voice, and never leave an old description sitting next to the new one — one current truth per item. Rewrite stale parts, add missing ones, remove what's gone. Then **bump "Last updated,"** add a **Known lag** flag if you couldn't fully reconcile this pass, and keep the invariants list current (a new invariant is often the most important thing a change introduces). If little actually moved, say so and make a minimal update rather than manufacturing churn.

For a large change, walk the sections a few at a time rather than dumping a full rewrite, so it stays reviewable.

## Adapting to your setup

Works for any prototype, any stack, any (or no) design system — it captures whatever is actually in the prototype, so your own components, tokens, and conventions come through on their own. A single HTML file, a React app, a Framer or v0 export, SwiftUI — the shape is the same; only the "File:" anchors change.

**Match your house style with an example.** Before writing, check for an `examples/` folder beside this skill; if one's there, mirror its structure, section names, and level of detail. That's the customization hook — a designer drops one of their own past handoffs into `examples/`, and every future note comes out shaped like theirs, no skill edits needed.

## Optional: Figma

If the prototype traces to a Figma source, you can cite node IDs (`Count badge 24525:10363`) so a dev can jump to the exact frame. Optional traceability — not required, and irrelevant if there's no Figma.
