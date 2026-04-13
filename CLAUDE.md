# CLAUDE.md — CareerSignal

Context for Claude Code sessions working on this project.

## What We're Building

**CareerSignal** — a community-driven job discovery platform for mid-career switchers. The core insight: people change careers by finding opportunities the community has already discussed and validated, not by spray-and-praying on LinkedIn.

The differentiator is community discussion *on* job listings — not alongside them. Every listing is enriched with peer intel (culture, interview process, red flags) before a user applies.

## Project Architecture

### Stack

- **Next.js 15** with App Router and Server Actions (no tRPC, no separate API layer)
- **PostgreSQL** via Drizzle ORM 0.38.x — Drizzle preferred over Prisma for serverless performance
- **Clerk** for auth (Google + LinkedIn OAuth pre-built; 10k MAU free)
- **Tailwind CSS 4** with shadcn/ui components (copy-owned, not a dependency)
- **TanStack Query** for client-side server state; RSC handles initial data fetching
- **Upstash Redis** for rate limiting and ephemeral cache (HTTP-based, serverless-safe)
- **Resend + React Email** for transactional email

### Directory Layout

```
CareerSignal/
├── src/
│   ├── app/                  # App Router pages + layouts
│   │   ├── (auth)/           # Login, signup, onboarding
│   │   ├── (feed)/           # Home feed, following feed
│   │   ├── jobs/             # Job detail + community thread
│   │   ├── companies/        # Company profiles + discussions
│   │   ├── profile/          # User profiles
│   │   └── api/              # API routes (non-Server Action endpoints)
│   ├── components/           # Shared UI
│   │   ├── feed/
│   │   ├── jobs/
│   │   ├── discussion/
│   │   └── profile/
│   ├── lib/
│   │   ├── actions/          # Server Actions (write paths)
│   │   ├── queries/          # Read-only data fetching
│   │   ├── db/               # Drizzle schema + client
│   │   └── search/           # Search index sync
│   └── types/                # Shared TypeScript types
```

### Key Architectural Decisions

- **Modular monolith**: All services live in the Next.js app. Extract to separate services only when a specific bottleneck demands it — not before.
- **Jobs own their community signal**: The `Job` entity stores denormalized `commentCount` and `lastActivityAt` — never COUNT on every feed render.
- **Shallow comment threading**: Max 2 levels (comment + one reply). No Reddit-style nesting — job discussion doesn't benefit from deep threads.
- **Pull-based feed at MVP**: Query follows + jobs + activity at read time. Migrate to hybrid push/pull only when feed queries exceed ~500ms p99.
- **Server Actions for mutations, RSC for reads**: No API routes for standard CRUD. Server Actions are already type-safe end-to-end.
- **Postgres as source of truth, Typesense for search**: Writes go to Postgres; a background worker syncs to Typesense within ~1s.

### Build Order

Always follow this sequence — each layer depends on the one before it:

1. Auth + Profiles (identity is required by everything)
2. Job Listings (the anchor entity)
3. Company Profiles (parent context for jobs)
4. Discussion (the core differentiator — prioritize over feed)
5. Follow Graph
6. Feed
7. Notifications
8. Search

## Design Preferences

### Visual Direction

- **Minimal** — no decorative elements, no gradients for the sake of gradients, no shadows unless they carry meaning
- **Generous white space** — let content breathe; tighter spacing signals hierarchy, not density
- **Typography-first** — Inter font across the product; weight and size carry the visual hierarchy, not color
- **Neutral palette** — muted backgrounds, high-contrast text, one accent color used sparingly

### Component Conventions

- Use shadcn/ui primitives as the base — own the source, customize freely
- Tailwind utility classes directly on components; no CSS modules
- Radix UI for accessible interactive primitives (dialogs, dropdowns, tabs)
- `cn()` utility for conditional class merging

### Layout Principles

- Feed cards: comfortable padding, clear job title hierarchy, community signal (comment count) as secondary metadata
- Job detail: generous line length (~65ch), clear section separation between listing and discussion thread
- Mobile-aware from the start even though web ships first

## Phase Status

| Phase | Goal | Status |
|-------|------|--------|
| 1. Foundation | Auth + job listing inventory | Not started |
| 2. Community | Threaded discussion on listings | Not started |
| 3. Discovery | Search, filters, community signal | Not started |

Planning docs live in `.planning/`. Requirements are in `.planning/REQUIREMENTS.md`. Roadmap is in `.planning/ROADMAP.md`.

## Out of Scope (do not implement)

- Direct messaging between users
- Quick apply / one-click apply
- AI resume builder
- Anonymous posting
- Native mobile app (web ships first)
- Deep comment threading beyond 2 levels
- Microservices before product-market fit
