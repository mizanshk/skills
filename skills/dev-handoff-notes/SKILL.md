---
name: dev-handoff-notes
description: >-
  Write or update a dev handoff doc — the annotation layer a designer hands
  engineering alongside a high-fidelity prototype. It states what EXISTS: the
  rules, states, edge cases and exact copy that aren't obvious from clicking
  through the prototype, what is mocked, and what is out of scope. It carries
  no rationale, no history, no attributions and no code internals. Use whenever someone wants to document a
  prototype for engineering ("write the handoff", "dev handoff notes", "update
  the handoff doc", "what does eng need to know about this").
---

# Dev Handoff Notes

## What this is

A dev handoff is the **annotation layer** for a prototype — the notes you would pin to the design file to say "this is a rule", "this is the edge case", "this string is exact", "this part is fake". It is read alongside the prototype by the developer building the real feature in the production codebase.

Two readers: the **developer**, and the developer's **AI coding assistant**, which gets the prototype plus this doc as build context.

**The bar: a developer who has never spoken to you can build the feature from the prototype and this doc, and nothing they build contradicts a decision you made.**

## What this is not

- **Not a design log.** The log holds history, alternatives, critique, who said what and why. None of that comes here. If a sentence explains *why*, it belongs in the log.
- **Not a rationale or alignment doc.** It doesn't justify decisions to stakeholders. It states them.
- **Not a spec of the prototype's code.** No file paths, selectors, component names, store shapes or repo invariants.
- **Not a description of the visible.** If it can be learned by clicking around, it isn't in here.

## The rules

1. **State what exists.** Every bullet is a rule, a behaviour, a state, an edge case, or an exact string. Nothing else.
2. **No rationale.** No "because", "so that", "in order to", no data points, no justification clauses. `The default window is Last 30 days.` — full stop.
3. **No history.** Never "replaced", "used to", "tried and reverted", "killed", "decided on <date>", "v1 → v2". Present tense only. The one comparison allowed is against **today's shipping product** when a delta is the fact itself (`Today's search misses partial words entirely.`).
4. **No attributions, no quotes from people.** No `(Name: "…")`, no names, no meeting references.
5. **The prototype is the source of truth and is never "wrong".** Never write "don't copy the prototype here". If the prototype contradicts a decision, that is a prototype bug — report it to the designer to fix, then update the doc. Parts outside the project's scope that aren't built go in **Not built in this prototype**, stated as absent.
6. **No code internals.** No paths, selectors, component or function names, hex values. Numbers appear only where the number *is* the rule — a breakpoint, a minimum touch target, a delay, a count.
7. **Only what isn't obvious.** Layout, hierarchy, panel widths, what sits left or right — visible, leave it out. Which click does what, what happens at zero, what a toggle's tooltip says, what a conflict looks like — in.
8. **Exact strings, verbatim.** Every user-visible string the developer must reproduce — labels, tooltips, toasts, empty states, confirms, banners — quoted exactly. Variants written as patterns: `` `<Active | Paused>, <expires <date> | never expires>` ``.
9. **One word per concept**, used everywhere. If the product calls it a *schedule*, it is never also a *window*, a *period* or *timing*.
10. **Logic before surfaces.** The model, precedence, conflict and permission rules come first; screens follow.
11. **Open questions are rare and are product questions.** Mark them inline as `⟨confirm: …⟩`. A question the *designer* can answer isn't a ⟨confirm⟩ — it's an inconsistency in the prototype to fix. A WIP project may carry a few; a finished one carries none.

## Structure

```
# <Project> — dev handoff notes

> **How to read this.** The prototype shows the behaviour: <URL>. The repo has
> the styles and code. This doc is the annotation layer: the rules, states and
> edge cases that aren't obvious from clicking through, what is mocked, and what
> is not built. Rationale is not in here.
>
> **Scope.** <What this covers. Which routes/areas in the repo belong to other
> work and should be ignored.>
>
> **Last updated:** <date>.        ← date only, never a changelog

## At a glance                       ← table: Area | What exists. One line per section.

## <The model / core logic>          ← objects, targets, precedence, permissions
## <Rules that span surfaces>        ← scheduling, conflicts, suggested states…

## <Surface 1>                       ← plain bullets; states, edge cases, exact copy
**Mocked:** <one line, only if something on this surface is simulated>
## <Surface 2> …

## Responsive                        ← table: Breakpoint | Change; then rules
## Motion rules                      ← policies, not values
## Accessibility                     ← what exists and is required in production
## Not built in this prototype       ← out of scope; absent, not simulated
## What the prototype fakes          ← simulated data and behaviour, consolidated
```

Per-surface bullets carry, as relevant: what each control does · every state (default, empty, zero, error, blocked) and its copy · edge cases (what happens at one, at many, at none) · exact strings · what's mocked on that surface.

**Not built** vs **Fakes**: *not built* is absent (no payment routing, no import flow); *fakes* is present but simulated (seeded analytics, a browser-storage store, a scripted customer). Keep them separate.

## Voice

Terse. Present tense. Bold the rule, then the specifics.

**Good:**
> - **Conflict = two codes on the same product with overlapping schedules.** Collection-vs-product is never a conflict. The picker refuses the second code (toast names the first); the detail page shows the banner and marks the rows.
> - **`Modified` = the last edit to the code itself.** Applying it to products, changing its schedule and redemptions do not update it.

**Not:**
> - The conflict rule is deliberately narrow because precedence already resolves collection-vs-product *(rationale)*
> - `Modified` replaced `Last redeemed`, which was customer activity *(history)*
> - Per the 08-25 review, stacking warnings were removed *(attribution + history)*
> - `CodesTable.tsx` sorts by `updatedAt` desc *(code internals)*
> - The panel is 400px wide, controls left, preview right *(visible)*

## How it works

**1. Find the doc.** `DEV-HANDOFF-NOTES.md` at the repo root, then `docs/`, then the working directory. Update in place — never a second file beside it. If none exists, create it at the repo root. Get the prototype URL from the designer if it isn't in the repo.

**2. Read the prototype — the code, not your memory.** Even when you built it this session. For each surface: every user-visible string (labels, tooltips, toasts, empty states, confirms, banners, placeholders), every state and its trigger, defaults, validation, what each control writes. Then the cross-cutting facts: persistence keys (→ Fakes), anything simulated or seeded (→ Fakes), routes and features that belong to other work (→ Scope), breakpoints and what changes at each (→ Responsive), animation names and reduced-motion handling (→ Motion), ARIA roles / focus handling / disabled vs unavailable (→ Accessibility). Check for loading and error states explicitly; if none exist, say nothing rather than invent them. Extract strings per file with two patterns — bare text lines starting with a capital, and quoted literals of six characters or more — and read anything ambiguous in context.

**3. Write** to the structure above. Logic first. Surfaces in the order a user meets them. Strings verbatim.

**4. Self-check** (below). Fix everything it catches before delivering.

**5. Report.** Tell the designer what went in, and — separately, not in the doc — every inconsistency you found in the prototype (two labels for one concept, a menu item that contradicts the rest, a stale string). Those are prototype fixes for the designer, not annotations.

## Updating an existing doc

Reconcile, never append. The prototype changed; find every bullet the change touches and rewrite it to the new truth. Never leave the old description next to the new one — one truth per item. Delete what's gone. Bump **Last updated** to the date, nothing more. When a prototype fix resolves a ⟨confirm⟩, delete the marker. If little changed, make the minimal edit; don't manufacture churn.

## Self-check before delivering

Run these on the finished file. Every hit is fixed, or sits inside quoted UI copy.

- **Attributions / history markers:** `grep -nE '\([A-Z][a-z]+(, [^)]*)?: \*?"|\b([Ss]aid|[Aa]greed|[Pp]ointed out|[Aa]ccording to|[Aa]sked for)\b|\b[Pp]er [A-Z][a-z]+|\b20[0-9]{2}-[0-9]{2}-[0-9]{2}\b|\b([Tt]he|[Dd]esign|[Ww]eekly|[Tt]hat|[Ll]ast) (review|critique|crit|meeting|1:1|sync)\b|\b[Tt]ranscript\b'` → 0 hits outside **Last updated** and quoted UI copy.
- **History:** `grep -nwE "replaced|used to|killed|reverted|tried|removed|originally|previously|no longer|decided"` → 0 hits (outside quoted UI copy).
- **Rationale:** `grep -nwE "because|so that|in order to|the reason|rationale|would|which is why"` → 0 hits.
- **Prototype-blame:** `grep -n "wrong\|Don't copy\|incorrect"` → 0 hits.
- **Code internals:** `grep -nE "\.tsx|\.ts\b|\.css|#[0-9a-fA-F]{6}|className|component"` → 0 hits.
- **Terminology:** pick the term for each concept; grep the alternatives → 0 hits outside quoted copy.
- **Strings:** every label, tooltip, toast, confirm and empty state the doc mentions is quoted verbatim and matches the code.
- **⟨confirm⟩:** each one is a product question, not a prototype inconsistency. Count them; a finished project has zero.
- **Visible-only bullets:** for each bullet ask "could the developer see this by clicking?" If yes, cut it.
- **Tables:** consistent column counts; **Last updated** is a date only.

## Examples

Mirror whatever sits in `examples/` beside this skill — structure, section names, voice, level of detail. If more than one example is there, prefer the most recent. Drop one of your own finished handoffs in.

## Edge cases

- **The project is WIP.** The doc will carry a few ⟨confirm⟩ markers and the prototype may have inconsistencies. Put the questions in the doc; put the inconsistencies in your report to the designer. Neither gets rationalised in the doc.
- **The prototype contains other work** (a vendored page, another project's routes, demo-only controls). Name them in **Scope** as out of scope, and list demo-only controls under **Fakes**. Don't document them.
- **A behaviour has no rule yet** (the prototype does something incidental). Ask the designer whether it's a rule. If it isn't, leave it out.
- **The designer wants to write it themselves.** Give them the verified facts and strings from step 2 and step back.
- **Someone asks for the "why".** Point them to the design log. If a `design-log` skill exists, that is where the material goes.
