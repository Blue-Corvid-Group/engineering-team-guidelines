# Engineering Team Guidelines

How we build software: our principles, patterns, and standards for working with agentic coding tools and each other.

This repo doubles as a Claude Code **skill**. The content is split across small files so a coding session loads only the part a task needs, instead of reading one large document every time.

| File | What's in it | When it's read |
|------|--------------|----------------|
| [`SKILL.md`](SKILL.md) | Thin router. The only part always in context. | Always (skill entry point) |
| [`principles.md`](principles.md) | Durable engineering principles, separation of concerns, layered architecture, working with AI agents, security, decision frameworks, checklists. | Any real code work |
| [`stack.md`](stack.md) | Preferred languages and frontend/backend/data/AI/deployment tools, described as *roles* rather than fixed versions. Volatile by design. | Only when making a tech-stack or architecture decision (triggers a fresh landscape check) |
| [`testing.md`](testing.md) | Testing philosophy, what to test vs. not, enforced coverage, periodic test review, smoke testing. | When writing/changing tests or finishing a feature |

---

## Setup (one time per machine)

The repo is the single source of truth. You install it once by symlinking it into your Claude Code skills directory and adding a short pointer to your global `CLAUDE.md`. Two steps.

### 1. Clone and symlink the skill

Clone anywhere you keep repos. The symlink uses the clone's actual location, so the path doesn't matter:

```bash
git clone https://github.com/Blue-Corvid-Group/engineering-team-guidelines.git
cd engineering-team-guidelines
ln -s "$(pwd)" ~/.claude/skills/engineering-guidelines
```

The symlink (rather than a copy) means `git pull` in this repo instantly updates the skill, no re-installing.

> On a machine where `~/.claude/skills/` doesn't exist yet, create it first: `mkdir -p ~/.claude/skills`.

### 2. Tell your global config to use it

A symlinked skill is dormant until something invokes it. Add this to your `~/.claude/CLAUDE.md` so engineering work consults it automatically:

```markdown
## Engineering work, read the team guidelines

Before writing, modifying, reviewing, refactoring, or architecting code,
consult the `engineering-guidelines` skill. It's split for token efficiency;
read only the part the task needs:

- principles.md — durable principles, architecture, security, decision
  frameworks. Read once per session the first time engineering work starts.
- stack.md — preferred tools. Read ONLY when making a tech-stack or
  architecture decision (it triggers a fresh landscape check, which is
  token-expensive, so skip it for routine work).
- testing.md — testing standards and smoke testing. Read when writing or
  changing tests, or finishing a feature.
```

Prefer not to edit your global config? You can instead invoke it explicitly in any session (`/engineering-guidelines`, or just ask Claude to consult the engineering guidelines). The auto-trigger above is the recommended path because it doesn't rely on remembering.

### 3. Verify

Start a Claude Code session and confirm the skill is registered:

```bash
ls -l ~/.claude/skills/engineering-guidelines   # should show the symlink → your clone
```

In a session, ask Claude to "consult the engineering guidelines" and confirm it reads `principles.md`. You're set.

### Staying current

`git pull` in your clone whenever you want the latest. Because it's symlinked, there's nothing else to do.

---

## Contributing

These guidelines improve when the team feeds findings back in. Open a PR.

### The stack evolves continuously

`stack.md` is meant to change. When a landscape check turns up a better default, a newly acceptable alternative, or a tool that should move to "avoid," PR it. The principles are durable; the stack is not. Describe tools by the *role* they fill, not by pinned versions or prices, those go stale in weeks.

### One rule keeps this lean: don't document what the harness owns

The most common way these guidelines rot is by drifting into restating how Claude Code itself works (which tools exist, which flags, which models, how subagents behave). That content is already covered by the harness, it goes stale every release, and it bloats what every session loads.

Before adding anything, ask: **is this ours, or is it the harness's?**

- Keep: our principles, our conventions, preferences that cut *against* agent defaults, and judgment the harness has no opinion on.
- Don't add: descriptions of harness mechanics, tool/flag names, model names, prices, or dated feature claims. Point at the harness's current guidance instead of freezing a snapshot.

When a preference also lives in a personal global `CLAUDE.md` (like secrets handling), keep one source of truth and have the other *point* at it rather than copying it.
