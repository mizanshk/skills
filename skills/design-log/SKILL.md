---
name: design-log
description: >-
  Maintains a running design log — a dated, verbatim record of decisions,
  iterations, critique, AI use, constraints, and rationale captured during
  ongoing design work. Records what was said and chosen, faithfully, so you (or
  an LLM you hand it to later) can reconstruct why a choice was made — for a
  stakeholder relitigating it, a teammate ramping up, your own recall months
  later, or source material for a case study. Use when the designer wants to log
  a decision, capture an iteration, record a critique session, note a
  constraint, flag AI use, or wrap up a session on an active project. Trigger
  phrases: "log this", "update the design log", "document this decision",
  "capture this", "add to the log", "design log update", "log this decision".
  Use this skill INSTEAD of writing log content directly — without its
  discipline an LLM aggregates, smooths, and loses the texture (exact values,
  verbatim quotes, precise verb choices, AI use, artifact links) that makes the
  log worth keeping.
---

# Design Log

Keeps a running log during ongoing project work — decisions, iterations, critique sessions, AI use, constraints, alternatives considered. Record what was said and chosen, not a summary of it, so the log can be *queried* later rather than trusted as a tidy recap.

The log is useful for:

- **Defending a decision** — a stakeholder reopens a choice, or a new PM asks "why is it built this way." The reasoning and the rejected alternatives are there, dated.
- **Team memory** — a teammate ramping onto the project can read why things are the way they are.
- **Your own recall** — you won't remember the exact curve, the exact objection, or why Option B died. The log will.
- **Case-study source material** — the raw material you harvest and filter when writing the case study later. It's the source, not the case study itself.

## Fidelity rules (what makes an entry well-formed)

These define a good entry. An entry that breaks them isn't a faithful record.

- **Quotes verbatim, in quotation marks, attributed and dated.** `"I'd have to go click that tile, even though technically I shouldn't have to" — [name], 2026-05-04 critique.` Never silently paraphrase a person into your own words.
- **If the exact wording isn't available, mark it `[paraphrased]`.** A labeled paraphrase is still queryable and can be back-filled with the real wording later; a paraphrase dressed as a quote can't be trusted and won't be caught.
- **Values as the literal string.** `450ms cubic-bezier(0.34, 1.56, 0.64, 1)`, not "an appropriate curve." The raw token is what a later query needs.
- **One entry per decision — preserve the count.** Three decisions in one session are three entries, not one paragraph. Collapsing them into "several decisions about X" is the most common and most damaging flattening.
- **Record, don't tidy.** Given a choice between the exact thing that was said and a cleaner summary of it, take the exact thing.

## What to capture

Capture liberally — when in doubt, log it. Every entry should fit one of the tags below; that's what keeps "liberal" meaning *every real decision, quote, value, and event* rather than a running stream of chatter. If it doesn't fit a tag, it isn't a log entry.

- **Decisions** — settled, parked, killed. What was decided, the reasoning, alternatives considered. Parked items marked "parked because X"; killed ones with why.
- **Iteration history** — v1 → v2 → v3, numbered, with what changed and why. Even a one-pass decision benefits from a brief "why I went straight here."
- **Direct quotes from named collaborators** — verbatim and attributed. Critique partners, engineers, stakeholders, users.
- **Specific values** — exact verbs, widths, colors, copy strings, animation curves, timing, tool names. Literal, never generic-ized.
- **Artifact links** — Figma files, screenshots, prototype URLs, PR links, thread links — inline at the relevant decision. *"v3 of the toast direction lives at <URL>."* Without them the log keeps what was decided but loses what it looked like.
- **AI use** — captured concretely. Actions (*"Used the model to survey error patterns across 12 tools; none use a banner"*) and raw pushback that mattered (*"Asked about Option B; pushback was 'we don't have that pattern anywhere in the product.' Killed it."*).
- **Constraints encountered** — engineering, brand, a11y, time, scope — and how the constraint shaped the work.
- **Trade-offs accepted** — what was given up to get what. Even small ones.
- **Process notes** — research patterns, critique stance, methods tried. Capture even before they're tied to a decision.
- **Meta-observations** — the designer's own pithy summaries and frame shifts. *"Visual symmetry was the bug, not the feature."*
- **Parked items, abandoned threads, rejected directions** — they matter for the record even if they never reach the case study.

Two things to keep out: **LLM filler** (*"It's important to note that…"*, *"Through user-centered design principles…"* — strip the opener; if nothing's left, drop the line), and **parked discussions written as settled decisions** (tag them `[Parked]`).

## How it works

Write the entries, then report what went in and where — no approval step in between. The four steps:

**1. Find the log.** Look for an existing one before creating it. Reasonable names: `LOG.md`, `DESIGN-LOG.md`, `design-log.md`, `PROJECT-LOG.md`. Check the current working location first, then an obvious `docs/`, `.planning/`, or logs folder if the setup has one.

- If logs live in a dedicated folder (e.g. a `design-logs/` directory with one subfolder per project), look there and match the current project by name, treating `_`, `-`, and spaces as equivalent. Only if that's how the setup is organized.
- **No filesystem?** In a chat interface with no files, the log is a document passed back and forth: the designer pastes in the current log, you return the updated version. Same discipline, different transport.
- If several candidates exist, list them with last-modified dates and ask which. If none exists, offer to create one and ask: what project (one-line scope), where it lives, and any notes to seed it.

Read the log fully if it exists — match its structure, voice, and date ordering.

**2. Pull loggable material from the conversation.** Scan back for anything matching a tag: decisions (*"let's go with X", "I'm killing Z", "parking this"*), attributed quotes, specific values and tool names, the *why* behind a choice, AI-use events, iteration moments, constraints, meta-observations. Capture the raw specifics, not a summary of them. If there's little worth logging, say so (*"Found N items — not much here. Log anyway, or skip?"*) rather than padding.

**3. Check against what's already logged.** For each item: already there? Can an existing point be enriched (add the quote, sharpen the rationale, add the value)? Net-new? Flag contradictions — often both versions belong, because the change *is* the iteration arc.

**4. Write and report.** Add entries in the log's format and voice, preferring targeted edits over rewriting the doc. Bump a "Last updated" line if one exists. Then summarize what went in and where, so the designer can spot-check without rereading the whole log.

## Log structure

Chronological at the top level, tagged within entries. Each entry begins with a date; newest at top or bottom is the designer's choice — stay consistent once chosen.

A typical entry:

```markdown
### 2026-05-04

**[Decision]** Toast slide-in animation locked. 450ms cubic-bezier(0.34, 1.56, 0.64, 1) spring entry, 220ms ease-in exit, 10s auto-dismiss. macOS-notification mental model. Considered a linear slide — rejected, felt like a state change rather than an arrival.

**[Critique]** [name]: *"I'd have to go click that tile, even though technically I shouldn't have to."* Cards read like decisions when they were already-on background state.

**[AI]** Used the model as a sparring partner on Option B. Asked for pushback; surfaced *"we don't have that pattern anywhere in the product."* Killed Option B.

**[Artifact]** v3 prototype: <URL>
```

Tags worth using inline: **[Decision]**, **[Critique]**, **[AI]**, **[Constraint]**, **[Iteration]**, **[Parked]**, **[Artifact]**, **[Process]**, **[Trade-off]**, **[Meta]**.

A short **Project context** preamble at the top (one paragraph: what this is, who it's for, the underlying primitive) is fine. Everything else lives in the dated flow.

The log doesn't need to be polished — **specific, dated, faithful, and complete.** Polish happens at case-study time.

## Quality bar

Six months later, a good entry lets you (or a model reading the log) reconstruct: what was decided and why, what alternatives were rejected, who said what (attributed and dated), the path the design took, the specific values chosen, how AI was used, and what constraints shaped the work.

Fails as *"we made several decisions about X."* Passes as: *"2026-05-04 — locked the toast slide-in: 450ms cubic-bezier(0.34, 1.56, 0.64, 1) spring entry, 220ms ease-in exit, 10s auto-dismiss. Reasoning: macOS-notification mental model — spring overshoot reads as physical, not mechanical. Considered a linear slide (rejected: reads as state change, not arrival). Engineering confirmed the timing budget was fine."*

## Edge cases

- **No log yet.** Offer to create; ask for scope, location, seed context.
- **Multiple candidates.** Present with last-modified dates; ask which.
- **Little material.** Say so and skip, rather than pad.
- **Material contradicts the log.** Surface it — *"earlier this was X, now Y; update, or keep both?"* Both often belong; the arc is the point.
- **Designer wants to write the entry themselves.** Step back — show what you found and let them write it. The job is that the material gets captured faithfully, not that you author it.
- **Designer asks for case-study work, not log work.** Different artifact: the case study is written later, framed for an audience and filtered hard. The log is the ongoing source. Hand off if a case-study skill exists.
