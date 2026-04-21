# Engineering Team Guidelines

How we build software -- our principles, patterns, and standards for working with agentic coding tools and each other.

---

## Table of Contents

- [Philosophy](#philosophy)
- [Principles We Follow](#principles-we-follow)
- [Separation of Concerns](#separation-of-concerns)
- [How We Think About Building](#how-we-think-about-building)
- [Preferred Stack](#preferred-stack)
- [Security](#security)
- [Working with AI Coding Agents](#working-with-ai-coding-agents)
- [Decision Frameworks](#decision-frameworks)
- [Testing and Quality](#testing-and-quality)
- [Quick Reference](#quick-reference)

---

## Philosophy

We bias toward action, simplicity, and pragmatism. Build what's needed, not what's interesting. Solve the problem in front of you with the least complexity that does the job well.

- **Minimal viable solution first.** Ask "what's the simplest thing that works?" before reaching for abstractions.
- **Reuse over rebuild.** Scan the codebase before writing new code. Extend what exists rather than creating duplicates.
- **Pragmatism over purity.** A hardcoded path that works beats an env-var abstraction you'll never port. Three similar lines of code is better than a premature abstraction.
- **Action over narration.** Run the build, start the server, execute the test. Don't describe what you're about to do -- just do it. Only pause for confirmation on destructive or irreversible actions.
- **Data over code.** When behavior needs to change, prefer changing data (config files, database rows, constants objects) over deploying new code. Business rules expressed as data are easier to audit, update, and extend.

---

## Principles We Follow

These are named principles from software engineering that shape how we work. We don't follow them dogmatically -- we follow them because they produce better software for our context.

### YAGNI (You Ain't Gonna Need It)

Don't build for hypothetical future requirements. Don't add error handling for scenarios that can't happen. Don't create abstractions for one-time operations. Don't add configurability to a simple feature. A bug fix doesn't need surrounding code cleaned up.

This applies to code you write and to code AI agents write on your behalf. Agents love to over-engineer -- give them clear constraints.

### KISS (Keep It Simple, Stupid)

Use the simplest tool that solves the problem. If `useState` is enough, don't reach for a state management library. If a JSON file works, don't build a database. If plain JavaScript covers it, don't add TypeScript. Complexity is a cost, not a feature.

This extends to dependency management: before installing a package, check if there's a lighter alternative, if you already have something similar, or if vanilla code would do the job. Every dependency is maintenance debt.

### DRY (Don't Repeat Yourself)

Before writing new functions, components, or utilities, scan the codebase for existing implementations. Reuse or extend what exists. If you're about to write similar code for the third time, extract it.

Pay special attention to UI patterns -- forms, tables, dashboards, modals, and control components often share 90%+ of their structure. Find the closest existing example and adapt it before building from scratch.

Global constants belong in a single location. Don't scatter magic values across files.

### Composition Over Inheritance

Build systems from small, focused units that compose together. Prefer composable state atoms over monolithic stores. Prefer functions that combine over class hierarchies that extend. Prefer hooks that wrap over base components that inherit.

This keeps individual pieces testable, replaceable, and understandable in isolation.

### Explicit Over Implicit

Prefer visible abstractions over hidden magic. If there's an auto-logout behavior on 403 responses, it should be in a documented interceptor -- not buried in framework middleware. If a service switches between storage backends, the detection logic should be readable, not inferred.

Code readers should be able to trace what happens by reading the code, not by understanding framework internals. When building service layers, name them clearly and document what they abstract.

### Convention Over Configuration (Local Conventions)

Establish project-level conventions and follow them consistently. If all models export `findAll`, `findById`, `create`, `update`, `remove` -- keep doing that. If all entity pages follow a split-panel editor pattern -- keep doing that. If all API routes are versioned under `/api/v1` -- keep doing that.

The value is in consistency, not in any particular convention. New code should look like existing code in the same project. Document conventions in the project's `CLAUDE.md` so both humans and agents follow them.

### Colocation (Organize by Feature, Not by Type)

Group related code together by what it does, not what it is. All approval logic in one service. All user-related models in one directory. All state atoms for a feature in one file.

"Where does the approval logic live?" should have one answer, not "spread across three controllers, two services, and a middleware."

---

## Separation of Concerns

Separation of concerns is how we think about building software at every level -- from how we structure files to how we design multi-service systems. The core idea: each piece of the system should have one job, and it shouldn't need to understand how other pieces do theirs.

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

Content, configuration, and business rules should live in data structures -- not embedded in the code that renders or executes them.

- **Content** lives in data files (JSON, YAML, markdown with frontmatter), not hardcoded in components. Edit the data to change what's displayed. The components are templates.
- **Configuration** lives in config files or database rows, not scattered as constants in service code. Adding a new AI model or a new vendor should be a config change, not a code change.
- **Business rules** expressed as data (scoring thresholds, approval settings, feature flags) are easier to audit and update than rules embedded in conditional logic.

This doesn't mean everything must be externalized. Inline a value if it's truly local to one place. Extract it when it's referenced from multiple places or when it represents a decision that might change independently of the code.

### State Separation

Client state and server state are different things and should be managed differently.

- **Server state** is data that lives on the backend and is cached locally -- use a server-state library (like a query cache) that handles staleness, refetching, and optimistic updates.
- **Client state** is UI-specific state like "which tab is selected" or "is the sidebar open" -- use lightweight, composable state primitives.
- **Persisted client state** (user preferences, last selection) belongs in localStorage with a project-specific key prefix to avoid collisions.

Don't put server data in client state stores. Don't manage UI toggles through API calls.

### Service Boundaries and Abstraction

When a system has multiple storage backends, external APIs, or processing tiers, create explicit service boundaries:

- Each external dependency should be accessed through a single service that abstracts its details. Consumers call `storageService.save()` without knowing if it's hitting a database, local storage, or cloud storage.
- AI model integrations should go through a configuration layer so you can swap models by changing config, not code.
- Multi-tier processing (e.g., frontend -> backend queue -> processing worker) should have clean handoff points with clear data contracts.

The test: can you swap the implementation behind a service boundary without changing its consumers? If yes, the boundary is clean.

### Frontend Architecture Patterns

For admin/CRUD interfaces, we follow a **split-panel editor pattern**:
- List panel on the left, editor panel on the right
- Fixed header/footer with scrollable content in the middle
- No modals for create/edit -- use the editor panel directly
- Selection persistence across sessions

For multi-service frontends, we follow a **monorepo with independent packages** pattern:
- Separate directories (`api/` + `web/` or `server/` + `client/`)
- Each side has its own dependency tree -- no shared node_modules trickery
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

The pattern: define an interface or contract, build a registry or config lookup, and let new variants plug in without modifying existing code. This is the Open/Closed Principle in practice -- open for extension, closed for modification.

### Lightweight Abstractions at Service Boundaries

Create abstractions where they earn their keep: at the boundaries between your code and external systems. A thin wrapper around a database client, a service layer between your UI and your API, a config lookup for AI model selection.

Don't create abstractions within a single layer. If a function is only called from one place, it doesn't need to be extracted into a utility. If a component has one layout, it doesn't need a layout abstraction.

### Convention-Driven Development

When you establish a pattern in a project, follow it everywhere in that project. Consistency is more valuable than local optimization. This is especially important when working with AI agents -- they learn from the patterns they see in your codebase.

Document conventions in your project's `CLAUDE.md`:
- How models/services export their functions
- How files and directories are organized
- How state is managed and where it lives
- How styling is applied
- How API routes are structured

If every model exports the same five functions, the next one should too. If every editor page follows the same panel layout, the next one should too.

---

## Preferred Stack

> **This section documents current preferences and will change over time.** The principles above are durable; the specific tools below are not. When a tool listed here no longer fits, swap it out -- the architecture should be tool-agnostic enough that switching is a configuration change, not a rewrite.

### The Stack Is Not Static

Our preferences reflect what works well for us today. The ecosystem moves, and a tool that was the right choice two years ago may be outclassed by something with better ergonomics, performance, bundle size, or community momentum today. Staying current is part of the job.

**Before making a non-trivial tech stack decision** -- picking a framework for a new project, choosing between two libraries for a core concern, committing to a pattern that will be hard to reverse -- take a fresh look at the landscape:

- **Scan the current conversation.** What are working engineers actually reaching for right now? Check release notes, migration guides, benchmark posts, and community discussion (Hacker News, Reddit, conference talks, well-regarded engineering blogs). Pay attention to what's gaining traction and what's quietly being abandoned.
- **Revisit our current preference.** Is the library listed below still actively maintained? Has its maintainer situation changed? Are users migrating away from it? A preference documented here is a default, not a mandate.
- **Consider emerging options seriously.** Newer tools that have earned strong adoption, solve real pain points, or meaningfully improve on incumbents deserve a look. Use the dependency evaluation criteria below -- don't adopt for novelty, but don't dismiss for it either.
- **Weigh switching cost honestly.** Changing the component library in one new project is cheap; changing it across a mature codebase is not. The bar for switching rises with blast radius.

**When you find something worth considering, surface it.** Bring it up in review, pilot it on a low-stakes project, and -- if it earns its place -- propose an update to this document. The guidelines improve when the team feeds findings back in. A new preference, a newly acceptable alternative, or a previously-preferred tool moving to the "avoid" list are all valid updates.

### How We Evaluate Dependencies

Every third-party library is a bet that someone else will maintain code you depend on. Evaluate that bet honestly:

- **Is it actively maintained?** Check commit recency, issue response times, and release cadence. A library that hasn't seen a commit in 18 months is a liability, not a time-saver. If it looks dead, avoid it -- even if the API is nice.
- **Is it stable and secure?** A well-engineered library with a small but active maintainer base can be better than a trendy one with a thousand stars and open CVEs. Quality of engineering matters more than hype.
- **Does it have community momentum?** Popularity isn't the goal, but it's a meaningful signal. Popular libraries attract contributors, get security patches faster, have better documentation, and are more likely to survive a maintainer stepping away. A library used by thousands of projects has thousands of reasons to stay maintained.
- **What's the blast radius if it dies?** A utility that formats dates is easy to replace. A UI component library that touches every page is not. The harder a dependency is to swap, the more you should favor established, well-maintained options over clever newcomers.
- **Is it the right size for the job?** Don't pull in a framework when you need a function. Check if there's a lighter alternative or if vanilla code would do.

The balance: don't chase trends, but don't ignore them either. A well-maintained project with strong community adoption is a safer bet than a technically superior one with a single maintainer and declining interest.

### Languages

| Preference | When to Use | Alternatives |
|-----------|-------------|--------------|
| **JavaScript** | Default for all new projects | TypeScript when the existing codebase or library ecosystem demands it |
| **Ruby** | Existing Rails API services | -- |
| **Python** | Data processing, ML pipelines | -- |

When type safety is needed in a JavaScript project, consider these before reaching for TypeScript: JSDoc type annotations, Zod for runtime validation, PropTypes for React components.

### Frontend

| Concern | Current Preference | Acceptable Alternatives |
|---------|-------------------|------------------------|
| **SPA framework** | React (via Vite) | -- |
| **Content/marketing sites** | Astro | -- |
| **Component library** | Mantine | Any actively maintained component library with strong community adoption (Chakra, Radix, shadcn). The key: use a library rather than building form inputs, modals, and data tables from scratch. Favor libraries with healthy contributor bases and responsive maintainers |
| **Styling** | Component library defaults + CSS variables | Tailwind, CSS modules. Avoid styled-components in new projects |
| **Client state** | Jotai | Zustand, or plain `useState` when state is local to one component. Avoid Redux |
| **Server state** | TanStack React Query | SWR, or any library that manages cache invalidation and background refetching |
| **Content schemas** | Zod | Yup, AJV. Use for both content collection validation and runtime input validation |

### Backend

| Concern | Current Preference | Acceptable Alternatives |
|---------|-------------------|------------------------|
| **API framework** | Express.js | Fastify, Hono. Rails for existing services |
| **Database** | PostgreSQL | SQLite for local development |
| **ORM/query builder** | Prisma (TypeScript projects), Knex (JS projects) | Raw SQL for complex queries is fine. Rails projects use ActiveRecord |
| **Auth** | JWT in localStorage | Supabase Auth, session cookies. Depends on the project |
| **Real-time** | ActionCable (Rails), WebSockets | SSE for simpler use cases |
| **API style** | REST with versioned routes (`/api/v1/`) | No GraphQL |
| **Serverless** | Vercel Functions | For secure API proxying where credentials can't be client-side |

### Data Patterns

| Concern | Current Preference | Notes |
|---------|-------------------|-------|
| **Flexible metadata** | JSONB columns with GIN indexes | Better than creating a new table for every variant. Use `store_accessor` in Rails |
| **Content management** | Astro Content Collections or markdown with frontmatter | For sites with structured content (blogs, programs, portfolios) |
| **AI model config** | Centralized config file mapping task types to provider + model | Swap models without code changes |

### Deployment

| Project Type | Current Preference | Acceptable Alternatives |
|-------------|-------------------|------------------------|
| **Static sites** | GitHub Pages | Vercel, Netlify, Cloudflare Pages |
| **SPAs** | Vercel | Heroku, Railway |
| **APIs** | Heroku | Railway, Render, Fly.io |
| **CI/CD** | GitHub Actions | -- |
| **Database migrations** | Forward-only, version-controlled | Never edit existing migrations. Never edit production directly |

---

## Security

### Secrets Management (Hard Rule)

**Never commit files that contain or could contain secrets.** No exceptions for "just this one variable" or "it's only a public URL."

#### Files that must be gitignored
- `.env`, `.env.*`, `.envrc` -- any environment file holding values
- Any file matching `*credentials*`, `*secrets*`, `*token*`, `*-key.json`, `*-key.pem`, `*.pfx`, `*.p12`
- Service account JSONs, OAuth tokens, database connection strings with passwords, private keys

#### Standard pattern
- Track `.env.example` with placeholder values: `OPENAI_API_KEY=sk-proj-your-key-here`
- `.gitignore` should explicitly allow examples: `!.env.example` after `.env.*`
- **New repos**: add gitignore entries on the very first commit, BEFORE any `.env` file exists

#### If you find a tracked secret file
Treat it as a security incident, even in a private repo:
1. Stop work immediately
2. `git rm --cached <file>` to untrack
3. Scrub history with `git filter-repo --path <file> --invert-paths`
4. Force push (`git push --force-with-lease`)
5. Rotate any exposed credentials -- assume they are compromised

### API Credential Security

When integrating third-party APIs, always ask: can this call happen client-side, or does it need a serverless proxy to protect credentials? Never expose API keys in client-side JavaScript. When in doubt, proxy through a serverless function.

---

## Working with AI Coding Agents

We use AI coding agents as core development tools. These guidelines help the team get consistent, high-quality results regardless of which agent or model is in use.

### Project Setup

Every project should have a `CLAUDE.md` (or equivalent instruction file) at the root. This is the agent's entry point for understanding your project. Include:

- **Tech stack** -- frameworks, languages, key dependencies
- **Architecture overview** -- how the project is structured, key directories, layer boundaries
- **Development commands** -- how to install, run, test, build, deploy
- **Conventions** -- naming patterns, file organization, state management approach
- **Critical rules** -- files not to modify, patterns that must be followed, known gotchas
- **Source-of-truth references** -- if content should come from a specific file, say so explicitly. Agents will hallucinate if they don't know where to look

### Guiding Agent Behavior

Agents are powerful but need constraints. Without them, they default to over-engineering.

- **Be specific about what you don't want.** "No TypeScript" is clearer than "prefer JavaScript." "No modals for editing" prevents the most common UI anti-pattern.
- **Call out protected files.** If certain files contain working functionality that shouldn't be touched, list them explicitly.
- **Document your patterns.** If the project has established conventions (editor layout, service layer structure, state management approach), describe them so the agent follows them rather than inventing its own.
- **Constrain scope.** "Fix the bug" is better than "fix the bug and clean up the surrounding code." Agents will happily refactor your entire file if you let them.
- **Require verification.** For AI-generated content (summaries, descriptions, analysis), require that all numerical data, statistics, and key metrics are preserved verbatim. Agents paraphrase and round numbers by default.

### Plan Execution

When given a plan to implement, execute steps in the specified order. Do not skip, reorder, or omit steps -- including infrastructure steps like branch creation and environment preparation.

- Treat plans as sequential checklists, not reference documents
- Setup steps come FIRST, before any code changes
- If a step seems unnecessary, flag it and ask before skipping

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
- What's the bundle size impact?
- Is it actively maintained? Check last commit, issue responsiveness, release cadence.
- Does it have meaningful community adoption? Stars alone don't matter, but usage and contributor count signal longevity.
- How hard would it be to replace if the project dies? The deeper the integration, the higher the bar for adoption.
- Is the landscape telling us something? If the guidelines list a preferred option for this concern, is it still the right default, or has the community moved on? Surface findings rather than silently drifting.

### Before Creating a New File

- Does something similar already exist in the codebase?
- Would this be a DRY violation?
- Should we extend an existing file instead?
- Does this follow the project's established organizational patterns?

---

## Testing and Quality

### Philosophy

Test the feature, not just the code. Type checking and test suites verify correctness at the code level, but they don't tell you if the feature works as intended. Start the dev server and use the feature in a browser before calling it done.

### Approach

- **Golden path first.** Make sure the intended workflow works end to end.
- **Edge cases second.** Test empty states, error states, boundary values.
- **Regression awareness.** When changing shared code, check that existing features still work.
- **Browser-based verification** for all UI work. Use automated browser tools (Playwright, Puppeteer) for repeatable testing and simulation.
- **Backend tests** for API logic and business rules (RSpec for Rails, or equivalent for your stack).

### Security Audits

- Track dependencies with automated tools (Dependabot, `npm audit`, `bundle audit`)
- Pre-commit hooks for catching common issues
- Treat any secret in version control as a security incident (see [Security](#security))

---

## Quick Reference

### New Project Checklist

1. Initialize repo with `.gitignore` that excludes secret files (BEFORE any `.env` exists)
2. Add `.env.example` with placeholder values
3. Create `CLAUDE.md` with stack, architecture, commands, and conventions
4. Set up file structure following established organizational patterns
5. Choose and configure component library and styling approach
6. Set up state management (server state cache + lightweight client state)
7. Configure deployment pipeline and CI

### Code Review Checklist

- [ ] No secrets committed or exposed in client-side code
- [ ] DRY -- no duplicated logic that should be extracted
- [ ] Data-driven where appropriate (constants files, config files, data files)
- [ ] Follows existing patterns and conventions in the codebase
- [ ] No unnecessary abstractions, speculative features, or scope creep
- [ ] Separation of concerns respected -- layers aren't crossing boundaries
- [ ] UI tested in browser, not just type-checked
- [ ] `CLAUDE.md` updated if architecture or conventions changed
- [ ] If the change introduces a new library or swaps a preferred one, the decision is justified against the current landscape and -- if broadly applicable -- these guidelines are updated
