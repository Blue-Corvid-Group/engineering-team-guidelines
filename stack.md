# Preferred Stack

> **You are reading this because you are making a tech-stack or architecture decision.** That is the only time to open this file. For routine code work, the principles in `principles.md` are enough.
>
> **First, do a fresh landscape check — then decide.** The tables below are a *snapshot of defaults*, not a mandate, and the ecosystem moves fast. Before committing to anything here:
>
> 1. **Check what's current right now.** Look at recent releases, migration guides, benchmark posts, and where working engineers are actually going (Hacker News, Reddit, conference talks, well-regarded engineering blogs). What's gaining traction? What's quietly being abandoned?
> 2. **Re-validate the listed preference.** Is it still actively maintained? Has the maintainer situation changed? Are users migrating away?
> 3. **Take emerging options seriously** if they've earned adoption or solve real pain, using the dependency criteria below.
> 4. **Weigh switching cost honestly.** New project: cheap to try something. Mature codebase: the bar rises with blast radius.
> 5. **Feed findings back.** If something has earned its place, update this file (PR to the team repo). A new preference, a newly acceptable alternative, or moving a tool to "avoid" are all valid updates. This file is meant to evolve continuously, not annually.
>
> The live check is token-expensive (web research), which is exactly why it's gated behind a real decision rather than run on every task.

---

## How We Evaluate Dependencies

Every third-party library is a bet that someone else will maintain code you depend on. Evaluate that bet honestly:

- **Is it actively maintained?** Check commit recency, issue response times, release cadence. A library untouched for 18 months is a liability, not a time-saver. If it looks dead, avoid it even if the API is nice.
- **Is it stable and secure?** A well-engineered library with a small but active maintainer base can beat a trendy one with a thousand stars and open CVEs. Quality of engineering matters more than hype.
- **Does it have community momentum?** Popularity isn't the goal, but it's a signal. Popular libraries attract contributors, get security patches faster, have better docs, and survive a maintainer stepping away.
- **What's the blast radius if it dies?** A date formatter is easy to replace. A UI component library that touches every page is not. The harder it is to swap, the more you favor established options over clever newcomers.
- **Is it the right size for the job?** Don't pull in a framework when you need a function.

The balance: don't chase trends, but don't ignore them. A well-maintained project with strong adoption beats a technically superior one with a single maintainer and declining interest.

---

## Languages

| Preference | When to Use | Alternatives |
|-----------|-------------|--------------|
| **JavaScript** | Default for all new projects | TypeScript when the existing codebase or library ecosystem demands it |
| **Ruby** | Existing Rails API services | — |
| **Python** | Data processing, ML pipelines | — |

When type safety is needed in a JavaScript project, consider these before reaching for TypeScript: JSDoc type annotations, Zod for runtime validation, PropTypes for React components.

## Frontend

| Concern | Current Preference | Acceptable Alternatives |
|---------|-------------------|------------------------|
| **SPA framework** | React (via Vite) | — |
| **Content/marketing sites** | Astro | — |
| **Component library** | shadcn/ui (Radix primitives + Tailwind, source copied into the project) | Mantine, Chakra, or any actively maintained library when shadcn doesn't fit (e.g. heavy data-table needs). The key: use a library rather than building form inputs, modals, and data tables from scratch. Favor healthy contributor bases and responsive maintainers |
| **Styling** | Tailwind CSS (paired with shadcn) | Component-library defaults + CSS variables, CSS modules. Avoid styled-components in new projects |
| **Client state** | Zustand | Plain `useState` when state is local to one component; Jotai when atomic/derived state is genuinely useful. Avoid Redux |
| **Server state** | TanStack React Query | SWR, or any library that manages cache invalidation and background refetching |
| **Content schemas** | Zod | Yup, AJV. Use for both content-collection validation and runtime input validation |

## Backend

| Concern | Current Preference | Acceptable Alternatives |
|---------|-------------------|------------------------|
| **API framework** | Fastify | Hono (esp. edge runtimes), Express for existing projects, Rails for existing services |
| **Database** | PostgreSQL | SQLite for local development |
| **ORM/query builder** | Prisma (TS projects), Knex (JS projects) | Raw SQL for complex queries is fine. Rails uses ActiveRecord |
| **Auth** | JWT in localStorage | Supabase Auth, session cookies. Depends on the project |
| **Real-time** | ActionCable (Rails), WebSockets | SSE for simpler use cases |
| **API style** | REST with versioned routes (`/api/v1/`) | No GraphQL |
| **Serverless** | Vercel Functions | For secure API proxying where credentials can't be client-side |

## Data Patterns

| Concern | Current Preference | Notes |
|---------|-------------------|-------|
| **Flexible metadata** | JSONB columns with GIN indexes | Better than a new table per variant. Use `store_accessor` in Rails |
| **Content management** | Astro Content Collections or markdown with frontmatter | For sites with structured content |
| **AI model config** | Centralized config file mapping task types to provider + model | Swap models without code changes |

## AI

The AI layer is a first-class part of our stack: it has its own client library, orchestration primitives, and observability concerns. Treat it like the data layer — pick the right tool for the boundary, abstract behind a service, tier model selection by task.

> This section describes *roles*, not specific model names or prices, on purpose: those age in weeks. Fill in the current models during your landscape check.

| Concern | Current Preference | Acceptable Alternatives |
|---------|-------------------|------------------------|
| **LLM client (TS/Node)** | [Vercel AI SDK](https://ai-sdk.dev) | Raw `@anthropic-ai/sdk` / `openai` only when the AI SDK lacks a needed primitive. Provider-agnostic call sites are the prerequisite for any cost/quality tiering |
| **LLM client (Python)** | Provider SDKs (`anthropic`, `openai`) directly | [Claude Agent SDK](https://code.claude.com/docs/en/agent-sdk/overview) when you need the same agent loop, tool execution, and context management that powers Claude Code. Check current subscription/billing terms if running it at volume |
| **Agent / workflow framework** | None by default | [Mastra](https://mastra.ai) (TS, built on the AI SDK) for durable workflows, first-class memory, or built-in evals. Skip frameworks for single-call use cases |
| **Durable execution around AI steps** | [Inngest](https://www.inngest.com) or [Vercel Workflow](https://vercel.com/docs/workflow) | Lightweight LLM calls inside durable, retriable, resumable steps. Use whenever an AI feature has more than one step that must survive a crash |
| **Model tiering** | Match model to task | A current frontier reasoning model for reasoning-heavy work; a current cheap/fast model for classification, extraction, dedup. The AI model config is where the per-task mapping lives. Look up the actual models and prices at decision time. |
| **Evals + observability** | Adopt when swapping models or scaling traffic | [Braintrust](https://www.braintrust.dev), [Langfuse](https://langfuse.com), [Helicone](https://www.helicone.ai). Don't adopt until you have the AI SDK in place and a real model swap on the horizon |
| **On-device LLM (mobile)** | [callstack/ai](https://github.com/callstackincubator/ai) on Expo + ExecuTorch | Same provider-agnostic shape as web. Treat as prototype-grade before committing |
| **MCP servers (local tools)** | Stdio MCP server colocated with the project | Expose read-only project data to Claude Code sessions. A hosted/remote MCP option exists if you ever need it; default to local |

**Default decision flow** for a new AI-shaped feature:

1. Single LLM call from TS/Node? AI SDK with config-driven model selection.
2. Needs streaming UI? AI SDK's React hooks.
3. More than one step that must survive a restart? Wrap in Inngest (or Vercel Workflow).
4. Needs memory, evals, or workflow primitives the above don't cover cleanly? Promote to Mastra.
5. Python, "Claude reads files and uses tools to produce an artifact"? Reach for the Claude Agent SDK rather than a custom orchestrator.
6. Mobile and needs offline? Prototype with `callstack/ai`.

Don't skip step 1 by reaching directly for an agent framework. Most AI features are a single well-prompted call, not an agent.

## Deployment

| Project Type | Current Preference | Acceptable Alternatives |
|-------------|-------------------|------------------------|
| **Static sites** | GitHub Pages | Vercel, Netlify, Cloudflare Pages |
| **SPAs** | Vercel | Heroku, Railway |
| **APIs** | Heroku | Railway, Render, Fly.io |
| **CI/CD** | GitHub Actions | — |
| **Database migrations** | Forward-only, version-controlled | Never edit existing migrations. Never edit production directly |
