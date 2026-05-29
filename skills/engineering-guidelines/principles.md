# Principles

The durable half of our engineering standards: how we think, regardless of stack. Tool choices live in `stack.md`; testing lives in `testing.md`.

---

## Philosophy

We bias toward action, simplicity, and pragmatism. Build what's needed, not what's interesting. Solve the problem in front of you with the least complexity that does the job well.

- **Minimal viable solution first.** Ask "what's the simplest thing that works?" before reaching for abstractions.
- **Reuse over rebuild.** Scan the codebase before writing new code. Extend what exists rather than creating duplicates.
- **Pragmatism over purity.** A hardcoded path that works beats an env-var abstraction you'll never port. Three similar lines of code is better than a premature abstraction.
- **Action over narration.** Run the build, start the server, execute the test, don't narrate it. Only pause for destructive or irreversible actions. (Operational detail in global CLAUDE.md → Execution Style.)
- **Data over code.** When behavior needs to change, prefer changing data (config files, database rows, constants objects) over deploying new code. Business rules expressed as data are easier to audit, update, and extend.

---

## Principles We Follow

Named principles that shape how we work. We don't follow them dogmatically, we follow them because they produce better software for our context.

### YAGNI (You Ain't Gonna Need It)

Don't build for hypothetical future requirements. Don't add error handling for scenarios that can't happen. Don't create abstractions for one-time operations. Don't add configurability to a simple feature. A bug fix doesn't need surrounding code cleaned up.

This applies to code you write and to code AI agents write on your behalf. Agents love to over-engineer, give them clear constraints.

### KISS (Keep It Simple, Stupid)

Use the simplest tool that solves the problem. If `useState` is enough, don't reach for a state management library. If a JSON file works, don't build a database. If plain JavaScript covers it, don't add TypeScript. Complexity is a cost, not a feature.

This extends to dependency management: before installing a package, check if there's a lighter alternative, if you already have something similar, or if vanilla code would do the job. Every dependency is maintenance debt.

### DRY (Don't Repeat Yourself)

Before writing new functions, components, or utilities, scan the codebase for existing implementations. Reuse or extend what exists. If you're about to write similar code for the third time, extract it.

Pay special attention to UI patterns. Forms, tables, dashboards, modals, and control components often share 90%+ of their structure. Find the closest existing example and adapt it before building from scratch.

Global constants belong in a single location. Don't scatter magic values across files.

### Composition Over Inheritance

Build systems from small, focused units that compose together. Prefer composable state atoms over monolithic stores. Prefer functions that combine over class hierarchies that extend. Prefer hooks that wrap over base components that inherit.

This keeps individual pieces testable, replaceable, and understandable in isolation.

### Explicit Over Implicit

Prefer visible abstractions over hidden magic. If there's an auto-logout behavior on 403 responses, it should be in a documented interceptor, not buried in framework middleware. If a service switches between storage backends, the detection logic should be readable, not inferred.

Code readers should be able to trace what happens by reading the code, not by understanding framework internals. When building service layers, name them clearly and document what they abstract.

### Convention Over Configuration (Local Conventions)

Establish project-level conventions and follow them consistently. If all models export `findAll`, `findById`, `create`, `update`, `remove`, keep doing that. If all entity pages follow a split-panel editor pattern, keep doing that. If all API routes are versioned under `/api/v1`, keep doing that.

The value is in consistency, not in any particular convention. New code should look like existing code in the same project. Document conventions in the project's `CLAUDE.md` so both humans and agents follow them.

### Colocation (Organize by Feature, Not by Type)

Group related code together by what it does, not what it is. All approval logic in one service. All user-related models in one directory. All state atoms for a feature in one file.

"Where does the approval logic live?" should have one answer, not "spread across three controllers, two services, and a middleware."

---

## Separation of Concerns

How we think about building software at every level, from how we structure files to how we design multi-service systems. The core idea: each piece of the system should have one job, and it shouldn't need to understand how other pieces do theirs.

### Layered Architecture

We build applications in distinct layers where each layer only talks to the one directly below it. The number of layers depends on the project's complexity, but the principle is the same: no skipping layers, and each layer has a clear responsibility.

**Frontend layers (typical React app):**

```
UI Components          → Render UI, handle user interaction
  ↓
Query/Mutation Hooks   → Manage server state, caching, loading states
  ↓
Service Layer          → One file per resource, CRUD functions, business logic translation
  ↓
HTTP Client            → Auth injection, base URL, error interceptors
  ↓
API
```

Components don't make HTTP calls. Services don't know about caching. The HTTP client doesn't know about business entities. Each layer does one thing.

**Backend layers (typical API):**

```
Routes/Controllers     → Parse request, validate input, return response
  ↓
Services               → Business logic, orchestration, authorization
  ↓
Models/Data Access     → Database queries, data transformation
  ↓
Database
```

Routes don't contain business logic. Services don't format HTTP responses. Models don't check permissions.

### Data Separate from Presentation

Content, configuration, and business rules should live in data structures, not embedded in the code that renders or executes them.

- **Content** lives in data files (JSON, YAML, markdown with frontmatter), not hardcoded in components. Edit the data to change what's displayed. The components are templates.
- **Configuration** lives in config files or database rows, not scattered as constants in service code. Adding a new AI model or a new vendor should be a config change, not a code change.
- **Business rules** expressed as data (scoring thresholds, approval settings, feature flags) are easier to audit and update than rules embedded in conditional logic.

This doesn't mean everything must be externalized. Inline a value if it's truly local to one place. Extract it when it's referenced from multiple places or when it represents a decision that might change independently of the code.

### State Separation

Client state and server state are different things and should be managed differently.

- **Server state** is data that lives on the backend and is cached locally. Use a server-state library (like a query cache) that handles staleness, refetching, and optimistic updates.
- **Client state** is UI-specific state like "which tab is selected" or "is the sidebar open." Use lightweight, composable state primitives.
- **Persisted client state** (user preferences, last selection) belongs in localStorage with a project-specific key prefix to avoid collisions.

Don't put server data in client state stores. Don't manage UI toggles through API calls.

### Service Boundaries and Abstraction

When a system has multiple storage backends, external APIs, or processing tiers, create explicit service boundaries:

- Each external dependency should be accessed through a single service that abstracts its details. Consumers call `storageService.save()` without knowing if it's hitting a database, local storage, or cloud storage.
- AI model integrations should go through a configuration layer so you can swap models by changing config, not code.
- Multi-tier processing (e.g., frontend → backend queue → processing worker) should have clean handoff points with clear data contracts.

The test: can you swap the implementation behind a service boundary without changing its consumers? If yes, the boundary is clean.

### Frontend Architecture Patterns

For admin/CRUD interfaces, we follow a **split-panel editor pattern**:
- List panel on the left, editor panel on the right
- Fixed header/footer with scrollable content in the middle
- No modals for create/edit, use the editor panel directly
- Selection persistence across sessions

For multi-service frontends, we follow a **monorepo with independent packages** pattern:
- Separate directories (`api/` + `web/` or `server/` + `client/`)
- Each side has its own dependency tree, no shared node_modules trickery
- Concurrent dev servers for local development

---

## How We Think About Building

### Progressive Disclosure of Complexity

Build the simple path first, then layer complexity on top. Systems should work in their minimal configuration and add behavior based on context.

- A storage service works with localStorage in development and switches to a cloud database in production.
- A scoring system shows "insufficient data" when there isn't enough data, instead of forcing users through a complex flow.
- An import system handles core fields by default and vendor-specific attributes as optional extensions.
- A component renders with sensible defaults and accepts props for customization.

This also applies to UI: show users what they need now and reveal deeper options as they engage.

### Configuration-Driven Extensibility

When you know the system will need to support new variants (new vendors, new AI models, new content types, new scrapers), design it so adding a variant is a configuration change, not a code change.

- New vendor? Add a config row with column mappings.
- New AI task? Add an entry to the models config file.
- New scraper type? Implement the interface and register it.

The pattern: define an interface or contract, build a registry or config lookup, and let new variants plug in without modifying existing code. This is the Open/Closed Principle in practice, open for extension, closed for modification.

### Lightweight Abstractions at Service Boundaries

Create abstractions where they earn their keep: at the boundaries between your code and external systems. A thin wrapper around a database client, a service layer between your UI and your API, a config lookup for AI model selection.

Don't create abstractions within a single layer. If a function is only called from one place, it doesn't need to be extracted into a utility. If a component has one layout, it doesn't need a layout abstraction.

### Convention-Driven Development

When you establish a pattern in a project, follow it everywhere in that project. Consistency is more valuable than local optimization. This is especially important when working with AI agents, they learn from the patterns they see in your codebase.

Document conventions in your project's `CLAUDE.md`:
- How models/services export their functions
- How files and directories are organized
- How state is managed and where it lives
- How styling is applied
- How API routes are structured

If every model exports the same five functions, the next one should too. If every editor page follows the same panel layout, the next one should too.

---

## Security

> The secrets hard-rule and its incident-response procedure live in the global `~/.claude/CLAUDE.md` ("Secret Files Rule"), which is the source of truth. This section states the principle and defers to it for the step-by-step.

**Never commit files that contain or could contain secrets.** No exceptions for "just this one variable" or "it's only a public URL." Gitignore `.env*`, `*credentials*`, `*secrets*`, `*token*`, `*-key.json`, `*-key.pem`, `*.pfx`, `*.p12`, service-account JSONs, and connection strings with passwords. Track `.env.example` with placeholders. Add gitignore entries on the very first commit, before any `.env` exists. If you find a tracked secret, treat it as a security incident and follow the global CLAUDE.md procedure (untrack, scrub history, force-push, rotate).

**API credentials:** never expose API keys in client-side JavaScript. When integrating third-party APIs, ask whether the call can happen client-side or needs a serverless proxy to protect credentials. When in doubt, proxy.

For dependency and audit tooling, see `testing.md` → Security Audits.

---

## Working with AI Coding Agents

We use AI coding agents as core development tools. These guidelines help the team get consistent, high-quality results regardless of which agent or model is in use.

### Project Setup

Every project should have a `CLAUDE.md` at the root, covering the usual ground (stack, architecture, commands, conventions, critical rules). The harness's `/init` produces a good starting skeleton; the point worth stressing because it's where agents fail:

- **Name your source-of-truth references explicitly.** If content must come from a specific file, say so. Agents hallucinate when they don't know where to look.

### Guiding Agent Behavior

Generic "give the agent context" advice is table stakes the harness already handles. These are the constraints that actually move our results, because they cut against agent defaults:

- **Be specific about what you don't want.** "No TypeScript" is clearer than "prefer JavaScript." "No modals for editing" prevents the most common UI anti-pattern.
- **Constrain scope.** "Fix the bug" beats "fix the bug and clean up the surrounding code." Agents will happily refactor your entire file if you let them.
- **Require verification of data.** For AI-generated content (summaries, descriptions, analysis), require that all numbers, statistics, and key metrics are preserved verbatim. Agents paraphrase and round by default.

### Plan Execution

When given a plan to implement, execute steps in the specified order. Do not skip, reorder, or omit steps, including infrastructure steps like branch creation and environment preparation.

- Treat plans as sequential checklists, not reference documents
- Setup steps come FIRST, before any code changes
- If a step seems unnecessary, flag it and ask before skipping

### Working with subagents

The harness owns the parallelism primitives (subagents, background tasks, agent teams) and how to choose among them, and it evolves faster than this document can. Defer to its current guidance rather than a frozen list here.

Two durable preferences that are *ours*, not the harness's, and worth stating because they cut against its defaults:

- **Token cost.** Prefer direct grep/Read over subagents for targeted work; subagents read files into their own context and cost several times more for the same lookup. We favor cheaper sessions over faster ones. (Full rule, including when subagents *are* worth it, lives in global CLAUDE.md → "Subagent usage.")
- **Match model to task difficulty.** Don't pay for the top tier on mechanical work, and don't starve a hard reasoning task to save pennies. Right-size, don't default.

---

## Decision Frameworks

### Before Building Anything New

1. **What specific problem am I solving?** If you can't state it in one sentence, you don't understand it yet.
2. **What's the minimal viable solution?** Start there.
3. **Does something already exist?** Check built-in features, existing packages, community tools. Search before building.
4. **Can a script solve this?** A 20-line bash script that works beats a 200-line abstraction that's "more maintainable."
5. **What's the maintenance cost?** Every dependency, every abstraction, every new file is something to maintain. Is the value worth the cost?
6. **Am I adding this because I need it or because it's interesting?** Be honest.

### Before Installing a Package

- Is there a lighter alternative? Would vanilla code do the job?
- Do we already have something similar in the project?
- What's the bundle-size impact?
- Is it actively maintained? Check last commit, issue responsiveness, release cadence.
- Does it have meaningful community adoption? Stars alone don't matter, but usage and contributor count signal longevity.
- How hard would it be to replace if the project dies? The deeper the integration, the higher the bar for adoption.
- Is the landscape telling us something? If `stack.md` lists a preferred option for this concern, is it still the right default, or has the community moved on? Surface findings rather than silently drifting.

### Before Creating a New File

- Does something similar already exist in the codebase?
- Would this be a DRY violation?
- Should we extend an existing file instead?
- Does this follow the project's established organizational patterns?

---

## New Project Checklist

1. Initialize repo with `.gitignore` that excludes secret files (BEFORE any `.env` exists)
2. Add `.env.example` with placeholder values
3. Create `CLAUDE.md` with stack, architecture, commands, and conventions
4. Set up file structure following established organizational patterns
5. Choose and configure component library and styling approach (see `stack.md`)
6. Set up state management (server-state cache + lightweight client state)
7. Configure deployment pipeline and CI

## Code Review Checklist

- [ ] No secrets committed or exposed in client-side code
- [ ] DRY — no duplicated logic that should be extracted
- [ ] Data-driven where appropriate (constants files, config files, data files)
- [ ] Follows existing patterns and conventions in the codebase
- [ ] No unnecessary abstractions, speculative features, or scope creep
- [ ] Separation of concerns respected — layers aren't crossing boundaries
- [ ] Tests added/updated for the change, and they're the right tests (see `testing.md`)
- [ ] UI tested in browser, not just type-checked
- [ ] `CLAUDE.md` updated if architecture or conventions changed
- [ ] If the change introduces a new library or swaps a preferred one, the decision is justified against the current landscape (see `stack.md`) and, if broadly applicable, the guidelines are updated
