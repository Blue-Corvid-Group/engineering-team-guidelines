---
name: engineering-guidelines
description: Blue Corvid Group / Jason's engineering standards for any code work under ~/code. Use BEFORE writing, modifying, reviewing, refactoring, or architecting code in any repo. Routes to three on-demand references — principles (always relevant), stack (architecture/tech decisions only), testing (when adding or touching tests). Read only the reference the task needs; do not load all three by default.
---

# Engineering Guidelines

Source of truth for how we build. Three reference files, loaded on demand — read the one the task calls for, not all of them.

| Read this | When |
|-----------|------|
| `principles.md` | Default for any real code work: principles (YAGNI/KISS/DRY/composition/explicit/colocation), separation of concerns, layered architecture, working with AI agents, security hard-rules, decision frameworks. Read once per session the first time engineering work starts. |
| `stack.md` | ONLY when making a tech-stack or architecture decision — picking a framework/library for a new project, choosing between two libraries for a core concern, or committing to a hard-to-reverse pattern. This file requires a fresh landscape check (current releases, what engineers are reaching for now); that research is token-expensive, so do not open it for routine work. |
| `testing.md` | When writing, changing, or reviewing tests, or finishing a feature (smoke test + coverage expectations). |

Do not restate these files back to the user. Apply them.

Hard rules that always apply (full detail in `principles.md` → Security): never commit secrets; gitignore `.env*` before the first commit; never expose API keys client-side. These also live in global CLAUDE.md — that file owns the secrets incident-response procedure; this skill defers to it.
