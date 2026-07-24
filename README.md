# Skills

Two skills I use for design work, built for [Claude Code](https://claude.com/claude-code) and any agent that reads the skills format.

Most of what matters on a project never gets written down: why a value is what it is, what got tried and killed, which parts of the prototype are real and which are faked with mock data. It's all clear while you're in the work. It's gone a few months later, or the moment someone else has to build from it. These two skills catch that detail at the two points it's easiest to lose: while the work happens, and when it hands off to engineering.

I wrote about the workflow behind these: [Build fast, forget faster](https://mizanshaikh.com/writing/build-fast-forget-faster).

## Install

```bash
npx skills@latest add mizanshk/skills
```

That pulls in both. Or copy either folder under `skills/` straight into `~/.claude/skills/` and skip the CLI.

## The skills

### design-log

A dated, verbatim record of decisions, iterations, critique, AI use, and constraints, kept while the work is happening. It holds what was actually said and chosen, not a tidy summary of it, so you can query it later instead of trusting a recap.

Fidelity is the whole point. Quotes stay in quotation marks, attributed and dated. Values stay literal: `450ms cubic-bezier(0.34, 1.56, 0.64, 1)`, not "a spring curve." Three decisions in one session stay three entries, not one paragraph. Left to itself, an LLM smooths all of that away, which is exactly the texture you were keeping the log for. The skill exists to stop it.

Useful for defending a decision a stakeholder reopened, ramping a teammate onto why things are the way they are, your own recall, and as raw material when you write the case study later.

Triggers on "log this", "update the design log", "document this decision".

### dev-handoff-notes

Everything a developer needs to build the real product from a high-fidelity prototype: interactions, every state, logic and rules, the data shape, accessibility, and what's mocked versus real. Written in the developer's language and anchored to the actual components and values in the prototype.

Where the log keeps history, this keeps what is. It describes the present and stays true to it: when the prototype changes, you rewrite the stale part instead of appending a note about the change. The section that earns its keep is mocked versus real, the line between where the prototype stops and production starts, each fake paired with what plugs in for it.

It also works as build context for an AI coding assistant. The prototype plus this doc is close to what an agent needs to build the real thing.

Triggers on "write up how this works for the devs", "handoff notes", "spec this for engineering".

## Why two

They're opposites, which is the reason they sit together. The log is chronological and keeps the why. The handoff is a present-tense spec and keeps the what. A project produces both kinds of knowledge and loses them the same way, so I catch them with separate tools.

## License

MIT. Use them, change them, ship your own.
