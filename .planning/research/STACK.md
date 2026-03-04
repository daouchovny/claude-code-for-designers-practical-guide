# Stack Research

**Domain:** Professional networking / community-driven job discovery platform
**Researched:** 2026-03-04
**Confidence:** MEDIUM-HIGH (core framework HIGH, auth ecosystem MEDIUM due to recent Auth.js/Better Auth merger)

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Next.js | 15.x (stable) | Full-stack React framework | App Router is the standard for new projects in 2025/2026. Server Components reduce client bundle for SEO-sensitive job listing pages. Built-in Server Actions replace boilerplate API routes for mutations (post discussion, follow user). Vercel hosts it natively. |
| React | 19.x | UI layer | Ships with Next.js 15. Server Components architecture aligns well with community-feed patterns where most content is read-only. |
| TypeScript | 5.x | Type safety | Non-negotiable for a platform where data schemas (job postings, profiles, discussions) are complex and interconnected. |
| PostgreSQL | 16.x | Primary database | The correct choice for this domain: relational data (users, jobs, follows, comments), full-text search via `pg_trgm` and `tsvector`, and row-level security. Supabase, Neon, and managed hosting all run Postgres natively. |
| Drizzle ORM | 0.38.x (stable) | Database access layer | Preferred over Prisma for this project. Drizzle is ~7kb, tree-shakeable, serverless-first, and generates SQL directly without an intermediate engine — critical for Vercel's serverless functions. Code-first schema definition in TypeScript keeps the codebase coherent. Note: v1.0-beta exists but use the latest stable 0.38.x for production. |
| Clerk | latest | Authentication + user management | For a social/community platform, auth is infrastructure not a differentiator. Clerk provides 10,000 MAUs free, pre-built OAuth (Google, LinkedIn — critical for professional networks), social login UI, user profile management, and SOC 2 Type II compliance. Eliminates weeks of auth engineering. Better Auth is a strong alternative if data residency is a constraint. |
| Tailwind CSS | 4.x | Styling | The 2025/2026 standard for Next.js. v4 drops the config file in favor of CSS-native `@theme` directives. Full support in shadcn/ui. |
| shadcn/ui | latest | Component library | Not a package you install — you copy source into your project and own it. 65k+ GitHub stars, adopted by Vercel and Supabase. Provides the Card, Dialog, Avatar, Badge, and Feed components this platform needs. Built on Radix UI for accessible primitives. |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Supabase Realtime (via `@supabase/realtime-js`) | 2.x | Live discussion updates | For new post notifications and real-time comment feeds. Supabase Realtime uses PostgreSQL's logical replication — no separate WebSocket infrastructure needed since you're already on Postgres. Use this instead of Pusher/Ably for early-stage. |
| TanStack Query | 5.x | Client-side server state | For data that needs background refetch, optimistic updates, or shared cache (e.g., discussion threads, notification counts). Works alongside RSC — RSC fetches initial data, TanStack Query handles subsequent client interactions. |
| Zod | 3.x | Schema validation | Used in Server Actions and API routes to validate user input (job posts, profile updates, discussion comments). Pairs with React Hook Form for form validation. |
| React Hook Form | 7.x | Form management | For the multi-step profile creation, job post submission, and comment forms. Integrates directly with Zod via `@hookform/resolvers`. |
| Resend + React Email | latest | Transactional email | For digest emails ("companies your community is discussing"), job alert notifications, and welcome flows. React Email lets you build templates as React components. 3,000 emails/month free. The de facto choice for modern Next.js stacks in 2025. |
| Upstash Redis (`@upstash/redis`, `@upstash/ratelimit`) | latest | Rate limiting + ephemeral cache | Serverless-native Redis (HTTP-based, no persistent connections). Use for: rate limiting discussion post endpoints, caching hot job feed queries, session-based features. Pay-per-request pricing works on Vercel serverless. |
| Meilisearch | 1.x | Full-text job search | For job title/company/skill search. PostgreSQL full-text search handles basic cases but lacks typo tolerance and relevance tuning. Self-hostable or Meilisearch Cloud. Start with Postgres FTS and migrate to Meilisearch when search quality becomes a pain point. |
| Sentry | 8.x | Error monitoring | The standard for production error tracking in Next.js. Has official Next.js SDK with App Router support, including Server Component and Server Action error capture. |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| Vercel | Hosting + preview deploys | Native Next.js platform. Preview deploys per PR are essential for reviewing community feed UI changes. Free tier is sufficient for early validation. |
| Drizzle Kit | Database migrations | Companion CLI to Drizzle ORM. Run `drizzle-kit push` for development, `drizzle-kit migrate` for production. |
| Drizzle Studio | Database browser | Visual table inspector, ships with Drizzle Kit. Replaces the need for a separate Prisma Studio. |
| ESLint + Prettier | Code quality | next lint includes ESLint 9 support out of the box. Add `eslint-config-prettier` to avoid conflicts. |
| Turbopack | Dev server bundler | Stable in Next.js 15 for development (`next dev --turbo`). Up to 96% faster HMR. Do not use for production builds yet. |

---

## Installation

```bash
# Bootstrap project
npx create-next-app@latest careersignal --typescript --tailwind --eslint --app --src-dir

# Core dependencies
npm install drizzle-orm @auth/drizzle-adapter postgres
npm install @clerk/nextjs
npm install zod react-hook-form @hookform/resolvers

# UI
npm install @radix-ui/react-dialog @radix-ui/react-avatar @radix-ui/react-dropdown-menu
# shadcn/ui components are added via CLI, not npm:
# npx shadcn@latest add button card avatar badge dialog

# Real-time + notifications
npm install @supabase/realtime-js
npm install resend react-email

# Caching + rate limiting
npm install @upstash/redis @upstash/ratelimit

# Client state
npm install @tanstack/react-query @tanstack/react-query-devtools

# Error monitoring
npm install @sentry/nextjs

# Dev dependencies
npm install -D drizzle-kit
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Clerk | Better Auth | When user data must be self-hosted (compliance, GDPR residency requirements, or budget sensitivity beyond 10k MAU). Better Auth is now the official successor to Auth.js for new projects. |
| Clerk | Supabase Auth | When you're already committed to the full Supabase platform and want a single vendor for DB + Auth + Storage. Supabase Auth is less polished for social/OAuth flows. |
| Drizzle ORM | Prisma | When your team is database-agnostic and wants automated migrations with a more Rails-like DX. Prisma 7 (late 2025) closed the performance gap substantially. |
| Supabase Realtime | Ably | When you need guaranteed delivery, complex routing, or >1000 concurrent connections in early phases. Ably is more reliable at scale but adds infra complexity and cost early. |
| Supabase Realtime | Pusher | Avoid Pusher — no Postgres integration, older architecture, and worse pricing than alternatives. |
| Meilisearch | Algolia | When search is a core differentiator and you have budget for enterprise pricing. Algolia is more mature but significantly more expensive at scale. |
| Meilisearch | Postgres FTS | For MVP only. Postgres `tsvector` + `pg_trgm` works for basic keyword search with no extra infrastructure. Migrate when typo tolerance complaints appear. |
| Vercel | Railway / Render | When you need persistent background workers (BullMQ + Redis) as first-class services. Vercel's serverless model doesn't support long-running processes; use Railway if you need background job processing beyond Upstash. |
| Resend | Nodemailer | Never — Nodemailer requires managing SMTP directly and has poor deliverability. Resend is purpose-built for modern Node/Next.js stacks. |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| NextAuth.js v4 | Officially in maintenance mode. Auth.js team has merged into Better Auth. No new feature development; security patches only. | Clerk (fastest) or Better Auth (self-hosted) |
| Auth.js v5 (next-auth@beta) | Still in beta as of March 2026. Never reached stable release. Ecosystem has moved toward Better Auth for new projects. | Clerk or Better Auth |
| Prisma v6 or older (pre-v7) | Rust query engine added cold-start latency on serverless — critical issue for Vercel. Prisma 7 (late 2025) fixed this, but Drizzle still has a smaller bundle and no compilation step. | Drizzle ORM |
| MongoDB | Community platforms with feeds, follows, and job listings are inherently relational. MongoDB's document model creates N+1 query problems and JOIN workarounds that fight the data model. | PostgreSQL |
| Firebase / Firestore | Vendor lock-in with no SQL escape hatch. Real-time features at the scale this platform needs are achievable with Supabase Realtime on Postgres. Firebase Auth is less capable than Clerk for professional OAuth flows (especially LinkedIn). | Supabase Realtime + Clerk |
| Redux | Overkill for this domain. Next.js 15 Server Components handle most "global" state server-side. TanStack Query handles async server state. Redux adds complexity with no benefit here. | TanStack Query + React context for lightweight local state |
| Webpack (manual config) | Next.js 15 ships Turbopack for dev (stable) and uses its own optimized Webpack config for prod builds. Do not eject or customize webpack manually. | Use Next.js defaults; Turbopack for dev |
| Planetscale MySQL | Planetscale pivoted to PostgreSQL-only ("Project Neki") — migration complexity is not worth it when Postgres is the default. | Neon or Supabase Postgres |
| tRPC | Valuable for large monorepos with separate frontend/backend teams. For a Next.js full-stack app using Server Actions, tRPC adds abstraction with no practical benefit — Server Actions are already type-safe end-to-end. | Next.js Server Actions |

---

## Stack Patterns by Variant

**If data residency / GDPR compliance is required:**
- Swap Clerk for Better Auth (self-hosted on your own Postgres) + use `better-auth` email/OAuth plugins
- Self-host Postgres on Hetzner (EU) or use Supabase with EU region
- Self-host Meilisearch on EU infrastructure

**If you need background job processing (email digests, recommendation engine):**
- Add BullMQ + Upstash Redis for job queues
- Or deploy a separate worker service on Railway alongside the Vercel Next.js app
- Vercel's `after()` API (stable in Next.js 15.1+) handles lightweight post-response tasks without a full worker queue

**If scaling search to 100k+ job listings:**
- Move from Postgres FTS to dedicated Meilisearch instance earlier
- Add Upstash QStash for indexing jobs asynchronously after creation
- Consider pgvector for semantic/AI-powered job matching once listings volume justifies it

**If mobile app follows web:**
- The Next.js API routes / Server Actions can be consumed directly by a React Native app
- Clerk has a React Native SDK — same auth provider works across web and mobile
- No architecture changes required; mobile becomes another client

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| Next.js 15.x | React 19.x, Node.js ≥ 18.18.0 | Minimum Node 18.18.0 required. React 19 is stable with App Router. |
| Drizzle ORM 0.38.x | PostgreSQL 14+, Node.js 18+ | Use `postgres` (node-postgres wrapper) as the driver, not `pg` directly |
| Tailwind CSS 4.x | shadcn/ui latest | shadcn confirmed Tailwind v4 support; uses `tw-animate-css` (replaces `tailwindcss-animate`) |
| Clerk latest | Next.js 15, React 19 | Clerk's Next.js SDK is regularly updated to track Next.js releases |
| TanStack Query 5.x | React 19 | v5.40.0+ supports streaming with RSC via dehydration of pending queries |
| Supabase Realtime 2.x | No Next.js version dependency | Used as a standalone JS client; not tied to Supabase hosting |

---

## Sources

- [Next.js 15 release blog](https://nextjs.org/blog/next-15) — Confirmed stable, React 19, Turbopack status — HIGH confidence
- [Auth.js joins Better Auth announcement](https://better-auth.com/blog/authjs-joins-better-auth) — Merger confirmed, Better Auth recommended for new projects — HIGH confidence
- [Auth.js v5 beta status discussion](https://github.com/nextauthjs/next-auth/discussions/13382) — Never reached stable, still beta — HIGH confidence
- [Drizzle ORM v1 roadmap](https://orm.drizzle.team/roadmap) — v1.0 still in beta; stable 0.38.x recommended — HIGH confidence
- [Prisma vs Drizzle 2026 comparison (makerkit.dev)](https://makerkit.dev/blog/tutorials/drizzle-vs-prisma) — Drizzle preferred for serverless Next.js; Prisma 7 closed the gap — MEDIUM confidence
- [Serverless PostgreSQL 2025 comparison (dev.to)](https://dev.to/dataformathub/serverless-postgresql-2025-the-truth-about-supabase-neon-and-planetscale-7lf) — Supabase preferred for product builders — MEDIUM confidence
- [Supabase blog: Postgres FTS vs alternatives](https://supabase.com/blog/postgres-full-text-search-vs-the-rest) — Postgres FTS viable for early stage — HIGH confidence
- [Clerk pricing page](https://clerk.com/pricing) — 10,000 MAU free confirmed — HIGH confidence
- [Upstash ratelimit library](https://github.com/upstash/ratelimit-js) — HTTP-based, designed for Vercel Edge — HIGH confidence
- [Resend 2025 new features](https://resend.com/blog/new-features-in-2025) — React Email 5.0 + Tailwind v4 support confirmed — HIGH confidence
- [shadcn/ui Tailwind v4 docs](https://ui.shadcn.com/docs/tailwind-v4) — Full v4 compatibility confirmed — HIGH confidence
- [TanStack Query v5 SSR guide](https://tanstack.com/query/v5/docs/react/guides/advanced-ssr) — RSC + streaming support confirmed — HIGH confidence

---

*Stack research for: CareerSignal — professional networking / community-driven job discovery platform*
*Researched: 2026-03-04*
