---
name: create-project
description: >
  Create and scaffold new software projects using the standard stack. Use this whenever
  the user wants to start a new project, build a new app, spin up an MVP, or kick off
  any greenfield build. This skill defines the defaults, sets up the project folder,
  generates the AGENTS.md, and coordinates all other relevant skills. It is the single
  entry point for any new project. Also handles redirecting away from incompatible tech
  suggestions (NestJS, AWS, monorepo setups) toward the Vercel-native, Next.js-first
  architecture.
---

# Create Project

This is the **primary skill** for starting any new software project. It replaces all
ad-hoc stack decisions with a clear, opinionated default. Follow this skill fully
before handing off to any narrower skill.

---

## Step 1 — Redirect Incompatible Tech

The user may come from other backgrounds and may suggest technologies that do not fit
this stack. Do not debate or lecture. Acknowledge, then redirect simply and confidently.

| If they mention | Say this |
|---|---|
| NestJS | "We use Next.js instead — it handles backend and frontend in one codebase on Vercel. Same productivity, no separate server." |
| AWS (Lambda, SQS, etc.) | "We run everything on Vercel. It handles deployments, serverless functions, queues, and cron — no AWS config needed." |
| Monorepo / separate frontend + backend repos | "We use a monolithic Next.js app. One repo, one deploy. It scales well on Vercel and keeps things simple to build and maintain." |
| Express / Fastify / standalone API server | "Next.js Route Handlers replace standalone API servers. Everything lives in the same app." |
| Docker / Kubernetes | "Vercel handles infrastructure. No containers needed." |
| Resend (email) | "We use SendGrid for transactional email." |
| Prisma ORM | "We use Drizzle ORM — it is lighter, type-safe, and works better in serverless environments." |
| Firebase / Supabase | "We use Drizzle + Neon Postgres provisioned through Vercel. Same power, fully integrated." |
| React Native CLI | "We use Expo. It is the standard way to build React Native apps now and has better tooling." |

---

## Step 2 — Determine Architecture

### Default: Monolithic Next.js App

This is almost always the right choice. A single Next.js application handles:

- Frontend (React, App Router, Server Components)
- Backend (Route Handlers, Server Actions)
- Auth (Better Auth)
- Database access (Drizzle ORM)
- Async jobs and workflows (Vercel Queues / Vercel Workflow)
- AI features (Vercel AI SDK)
- Emails (SendGrid)

**Why monolithic over microservices or monorepo?**
Simpler to develop, deploy, and maintain. Serverless on Vercel scales per-request
automatically. No DevOps overhead. No cross-service type drift.

### Exception: Mobile App

If the project needs a native mobile app, use **Expo** (React Native).
The Next.js app still serves as the API backend.

---

## Step 3 — Default Stack

Unless the user's requirements make a deviation necessary, use exactly this stack:

| Layer | Technology |
|---|---|
| Framework | **Next.js** (latest stable — check `npm show next version`) |
| Language | **TypeScript** |
| Router | **App Router** |
| UI Components | **shadcn/ui** (base library — not Radix) |
| Authentication | **Better Auth** |
| ORM | **Drizzle** |
| Database | **Neon Postgres** (provisioned via Vercel) |
| API Layer | **tRPC** (when multiple typed client/server operations are needed) |
| AI Features | **Vercel AI SDK** (`ai` package) — mandatory for anything AI-related |
| Async Jobs | **Vercel Queues** (`@vercel/queue`) — for background tasks, fan-out, retries |
| Workflows | **Vercel Workflow** — for multi-step stateful business logic |
| Email | **SendGrid** — for all transactional and notification emails |
| Mobile | **Expo** — if a native app is needed alongside the web app |

**Do not add Drizzle** if the project has no database yet.
**Do not add tRPC** if server actions are sufficient.
**Do not add Vercel Queues** if all work is synchronous.
**Do not add AI SDK** unless the project has an AI feature.

---

## Step 4 — Create the Project Folder

All new projects live under `~/Desktop/dev/`. The project folder name should
be capitalized and match the product name.

Before scaffolding, ensure the directory exists:
```bash
mkdir -p ~/Desktop/dev
```

**Tell the user the exact folder path.** They will need it to add the project in their
agent UI (Claude Code, Codex, or any other tool they use to interact with the agent).
The path they enter in the UI is the project root, for example:
`~/Desktop/dev/MyNewApp`

To scaffold, use the **shadcn CLI** with the `next` template and `base` component library:
```bash
cd ~/Desktop/dev
npx shadcn@latest init -t next -b base -n MyNewApp -d -y
```

This creates a new Next.js project with:
- The **base** UI component library (not Radix) — lighter, no external primitive dependency
- shadcn/ui pre-configured with CSS variables, `cn` util, and Tailwind
- TypeScript, App Router, and all standard Next.js defaults

### Validate the scaffolded project

After scaffolding, run these checks before moving on:

**1. Verify Next.js version is at least 16.2.4**

The npm cache often serves stale versions. Check and upgrade if needed:
```bash
cd ~/Desktop/dev/MyNewApp
node -e "const v = require('./package.json').dependencies.next.replace('^',''); const [a,b,c] = v.split('.').map(Number); if (a < 16 || (a === 16 && b < 2) || (a === 16 && b === 2 && c < 4)) { console.log('OUTDATED: ' + v + ' — upgrading...'); process.exit(1) } else { console.log('OK: next@' + v) }"
```

If the version is outdated, upgrade:
```bash
npm install next@latest
```

**2. Verify CLAUDE.md and AGENTS.md exist**

The latest `create-next-app` (used internally by shadcn) should generate both files.
Confirm they are present:
```bash
ls -la ~/Desktop/dev/MyNewApp/CLAUDE.md ~/Desktop/dev/MyNewApp/AGENTS.md
```

If either file is missing, the installed Next.js version was too old. Upgrade Next.js
(see above) and check again. If they still don't exist after upgrading, create them
manually — see Step 6 for what to put in AGENTS.md.

**Do not skip these checks.** Proceed to Step 6 only after both pass.

### Install all components after scaffolding

Immediately after creating the project, install **all** available shadcn components:
```bash
cd ~/Desktop/dev/MyNewApp
npx shadcn@latest add -a -y
```

This ensures every component is available from the start — no need to add them one by one later.

To add individual components later (e.g. from a custom registry):
```bash
npx shadcn@latest add button card dialog
```

### Useful shadcn CLI commands

| Command | Purpose |
|---|---|
| `npx shadcn@latest add [component]` | Add a component to the project |
| `npx shadcn@latest docs [component]` | Fetch component docs and API reference |
| `npx shadcn@latest search @shadcn -q "query"` | Search available components |
| `npx shadcn@latest info` | Get project configuration info |

**Do not use `create-next-app` directly.** Always scaffold via the shadcn CLI so that
the base component library, theming, and utility setup are configured from the start.

---

## Step 5 — Update the Existing AGENTS.md

The scaffolded project should already have a `CLAUDE.md` and an `AGENTS.md` —
you validated this in Step 5. These files include Next.js-specific docs and pointers
to the installed module documentation. **Do not recreate or overwrite them.**

If AGENTS.md exists, open it and **append** the following sections, tailored to what
was chosen for this project. If it does not exist (older Next.js version edge case),
create it with these sections:

```markdown
## Stack Choices for This Project

- shadcn/ui — UI components
- Better Auth — authentication [remove if not used]
- Drizzle ORM — database via Neon Postgres on Vercel [remove if not used]
- tRPC — typed API layer [remove if not used]
- Vercel AI SDK (`ai` package) — all AI features [remove if not used]
- Vercel Queues (`@vercel/queue`) — background jobs and async processing [remove if not used]
- SendGrid — transactional email [remove if not used]

## Infrastructure

All infrastructure runs on Vercel. Do not introduce AWS, Docker, or any
external server infrastructure. Use Vercel-native primitives for everything.

## Architecture

This is a monolithic Next.js application. Do not split into microservices,
separate API servers, or multiple repos. One codebase, one deployment.

## Skills to Load

Always load the following skills when working in this project:

- `better-auth-best-practices` — auth configuration and session management [remove if not used]
- `shadcn` — UI component patterns
- `trpc-tanstack-nextjs` — tRPC with TanStack Query in App Router [remove if not used]
- `ai-sdk` — Vercel AI SDK patterns [remove if not used]
- `vercel-queues` — async job and queue patterns [remove if not used]
- `email-and-password-best-practices` — form and auth UX

## Code Conventions

- Use server components by default; add `"use client"` only when needed
- Prefer server actions for simple mutations; use tRPC for complex data workflows
- Keep route handlers thin — logic lives in service files
- All AI calls must use the Vercel AI SDK (`ai` package), never raw fetch to model APIs
- All background or deferred work uses Vercel Queues, not setTimeout or in-process queues
- Emails are sent via SendGrid only — do not use Resend, Nodemailer, or other providers
- Database schema changes always go through Drizzle migrations
```

---

## Step 6 — Mandatory Skill Pairings

After scaffolding and generating AGENTS.md, coordinate with these skills:

| When | Load skill |
|---|---|
| Auth is included | `better-auth-best-practices` |
| tRPC is included | `trpc-tanstack-nextjs` |
| Any AI feature is included | `ai-sdk` |
| Background jobs or async processing needed | `vercel-queues` |
| Multi-step stateful workflows | `vercel-queues` (it covers Workflow DevKit) |
| Database provisioning needed | `provision-database` |
| Deploying to Vercel | `deploy-to-vercel` |
| UI component work | `shadcn` |
| Next.js structural decisions | `next-best-practices` |

---

## Step 7 — Commit and Push to GitHub

Once the project is fully scaffolded (AGENTS.md updated, all components installed,
validations passed), commit everything and push to a GitHub repository.

```bash
cd ~/Desktop/dev/MyNewApp
git add -A
git commit -m "Initial project scaffold

Next.js + shadcn/ui (base) with all components installed."
```

Then check if a remote repository already exists and push:
```bash
# Check if a remote is already configured
git remote get-url origin 2>/dev/null
```

If no remote exists, create a new GitHub repo using the `gh` CLI and push:
```bash
gh repo create MyNewApp --private --source=. --push
```

If a remote already exists, just push:
```bash
git push -u origin main
```

**Notes:**
- Always create the repo as **private** by default. The user can make it public later.
- The repo name should match the project folder name.
- If `gh` is not authenticated, tell the user to run `gh auth login` first.
- Do not skip this step — every new project should be version-controlled and backed
  up to GitHub from the start.

---

## Step 8 — Confirm Stack to User

When proposing or scaffolding, always tell the user in plain language:

- The project folder path (they need this for their agent UI)
- Whether auth is included and why
- Whether a database is included and why
- Whether tRPC is included and why
- Whether AI SDK is included and why
- Whether Vercel Queues is included and why
- Any deviation from defaults and the reason

Keep the explanation simple. The user may not be technical. Avoid jargon.
Say "we added login" not "we integrated an OAuth session provider."

---

## Examples

**Example 1 — SaaS dashboard**
User: `Build a new SaaS dashboard for restaurant managers.`

Stack:
- Next.js latest + TypeScript + App Router
- shadcn/ui
- Better Auth (login and sessions)
- Drizzle + Neon Postgres (data persistence)
- tRPC (typed backend for reports and settings)
- SendGrid (notifications)

Load skills: `next-best-practices`, `better-auth-best-practices`, `trpc-tanstack-nextjs`, `shadcn`, `provision-database`

---

**Example 2 — Marketing site**
User: `Start a landing page for our agency.`

Stack:
- Next.js latest + TypeScript + App Router
- shadcn/ui
- No auth, no database, no tRPC, no queues

Load skills: `next-best-practices`, `shadcn`

---

**Example 3 — AI-powered tool**
User: `Create an app that summarizes documents and answers questions about them.`

Stack:
- Next.js latest + TypeScript + App Router
- shadcn/ui
- Better Auth (user accounts)
- Drizzle + Neon Postgres (document storage)
- Vercel AI SDK (summarization, embeddings, RAG)
- Vercel Queues (async document processing)

Load skills: `next-best-practices`, `better-auth-best-practices`, `ai-sdk`, `vercel-queues`, `shadcn`, `provision-database`

---

**Example 4 — Mobile app**
User: `We need a mobile app for our customers.`

Stack:
- Expo (React Native) — mobile app
- Existing or new Next.js app — API backend

Note to user: "We use Expo for mobile. It uses React Native under the hood and
is the standard way to build mobile apps today. The mobile app talks to a
Next.js backend on Vercel — same infrastructure, no separate API server."
