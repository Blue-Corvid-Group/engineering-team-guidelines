# Engineering Team Guidelines

How we build software: our principles, patterns, and standards for working with agentic coding tools and each other.

This repo is a Claude Code **plugin** that bundles the `engineering-guidelines` skill. The skill is split across small files so a coding session loads only the part a task needs, instead of reading one large document every time. Claude consults it automatically when it detects engineering work (per the skill's description); no per-session token cost when you're not coding.

| File | What's in it | When it's read |
|------|--------------|----------------|
| [`skills/engineering-guidelines/SKILL.md`](skills/engineering-guidelines/SKILL.md) | Thin router. The only part always in context when the skill is active. | Skill entry point |
| [`principles.md`](skills/engineering-guidelines/principles.md) | Durable engineering principles, separation of concerns, layered architecture, working with AI agents, security, decision frameworks, checklists. | Any real code work |
| [`stack.md`](skills/engineering-guidelines/stack.md) | Preferred languages and frontend/backend/data/AI/deployment tools, described as *roles* rather than fixed versions. Volatile by design. | Only when making a tech-stack or architecture decision (triggers a fresh landscape check) |
| [`testing.md`](skills/engineering-guidelines/testing.md) | Testing philosophy, what to test vs. not, enforced coverage, periodic test review, smoke testing. | When writing/changing tests or finishing a feature |

---

## Install (one time per machine)

Install it as a plugin from this repo's marketplace. No clone required, and updates propagate without re-installing.

```
/plugin marketplace add Blue-Corvid-Group/engineering-team-guidelines
/plugin install engineering-guidelines@blue-corvid-group
/reload-plugins
```

That's it. The skill auto-surfaces when Claude detects code work, because its description says to consult it before writing, reviewing, or architecting code. You do **not** need to edit your global `CLAUDE.md`.

### Auto-update (recommended)

To get stack and guideline changes automatically at session start, add this to your `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "blue-corvid-group": {
      "source": { "source": "github", "repo": "Blue-Corvid-Group/engineering-team-guidelines" },
      "autoUpdate": true
    }
  }
}
```

With `autoUpdate` on, Claude Code refreshes the plugin at startup, so a merged change to `stack.md` reaches you on your next session. Without it, pull updates manually:

```
/plugin marketplace update blue-corvid-group
/reload-plugins
```

### Verify

```
/plugin
```

`engineering-guidelines` should appear as installed. Ask Claude to "consult the engineering guidelines" in a session and confirm it reads `principles.md`.

### Stronger triggering (optional)

The skill description handles triggering for most work. If you want a harder guarantee on a specific project, add a line to that project's `CLAUDE.md`: *"Consult the `engineering-guidelines` skill before code work."* The skill content still loads only on demand, so this stays token-cheap.

---

## Contributing

These guidelines improve when the team feeds findings back in. Open a PR. Bump `version` in `.claude-plugin/plugin.json` when you want the change to ship as a new version to teammates with auto-update on.

### The stack evolves continuously

`stack.md` is meant to change. When a landscape check turns up a better default, a newly acceptable alternative, or a tool that should move to "avoid," PR it. The principles are durable; the stack is not. Describe tools by the *role* they fill, not by pinned versions or prices, those go stale in weeks.

### One rule keeps this lean: don't document what the harness owns

The most common way these guidelines rot is by drifting into restating how Claude Code itself works (which tools exist, which flags, which models, how subagents behave). That content is already covered by the harness, it goes stale every release, and it bloats what every session loads.

Before adding anything, ask: **is this ours, or is it the harness's?**

- Keep: our principles, our conventions, preferences that cut *against* agent defaults, and judgment the harness has no opinion on.
- Don't add: descriptions of harness mechanics, tool/flag names, model names, prices, or dated feature claims. Point at the harness's current guidance instead of freezing a snapshot.

When a preference also lives in a personal global `CLAUDE.md` (like secrets handling), keep one source of truth and have the other *point* at it rather than copying it.

---

## Repository layout

```
.claude-plugin/
  plugin.json         # plugin manifest
  marketplace.json    # marketplace catalog (lets teammates add this repo directly)
skills/
  engineering-guidelines/
    SKILL.md          # router
    principles.md
    stack.md
    testing.md
README.md
```
