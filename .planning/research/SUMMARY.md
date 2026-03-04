# Project Research Summary

**Project:** CareerSignal
**Domain:** Community-driven job discovery platform for mid-career switchers
**Researched:** 2026-03-04
**Confidence:** MEDIUM-HIGH

## Executive Summary

CareerSignal is a two-sided community platform where the value proposition is social intelligence layered onto job listings — not the listings themselves. Experts building in this domain (Levels.fyi for salary transparency, Blind for anonymous professional intel, Fishbowl before the Glassdoor acquisition) converge on a consistent approach: job content is the anchor, community signal is the differentiator, and identity architecture (how anonymous vs. verified users are) determines whether the community produces actionable intel or drifts into noise. The recommended implementation is a Next.js 15 full-stack monolith with PostgreSQL as the relational core, Clerk for auth, Drizzle ORM for database access, and Supabase Realtime for discussion activity — a stack optimized for fast iteration on Vercel's serverless infrastructure without the cold-start latency penalties that plagued earlier Prisma-on-serverless patterns.

The build sequence is architecturally constrained: Auth and Profiles must come first because every community action requires identity. Job Listings and Company Pages are the anchor entities everything else attaches to. Discussion Threads on those listings are the core differentiator and must be prioritized over feed personalization and follow graphs. The critical architectural decisions — shallow 2-level comment threading, denormalized community signal on the Job entity, and pull-based feed assembly — should be locked in at the data model stage, not retrofitted later. The biggest non-technical risk is the cold-start problem: the differentiating feature (community-vetted signal on listings) is invisible until there are enough community members creating signal, which makes pre-launch community seeding a product requirement, not a marketing afterthought.

The top risks are all known and well-documented from comparable platforms: (1) launching to an empty community and watching users bounce because the product looks like a plain job board, (2) allowing audience scope to creep beyond mid-career switchers and losing the niche identity that makes the platform trustworthy to its core users, and (3) letting employer monetization erode community trust by giving paying employers influence over community content. All three are preventable if caught at Phase 1 design, and extremely costly to reverse once they've taken hold. Building in expiry rules for stale listings, structured intel prompts for community discussion, and hard employer/community separation into the data model and product policy from day one is non-negotiable.

## Key Findings

### Recommended Stack

The stack is centered on Next.js 15 with the App Router, which provides Server Components for fast initial page loads on SEO-critical job listing pages and Server Actions for type-safe mutations (posting comments, following companies) without boilerplate API routes. PostgreSQL 16 is the correct database choice for this domain — it handles the relational data model (users, jobs, follows, comments), supports full-text search via `pg_trgm` and `tsvector` for early-stage job search, and has native tooling in Supabase and Neon. Drizzle ORM (stable 0.38.x) is preferred over Prisma for its serverless-first design: smaller bundle, no Rust query engine cold-start latency on Vercel, and code-first TypeScript schema definition that keeps the codebase coherent with the rest of the project.

Auth is infrastructure, not a differentiator: Clerk provides LinkedIn and Google OAuth (critical for a professional platform), 10,000 MAU free, and SOC 2 Type II compliance, eliminating weeks of auth engineering. The notable ecosystem shift: NextAuth.js v4 is in maintenance mode, Auth.js v5 never reached stable, and the community has consolidated on Clerk (managed) or Better Auth (self-hosted) for new projects. Supabase Realtime handles live discussion updates without separate WebSocket infrastructure, since the project is already on Postgres. Upstash Redis (HTTP-based, pay-per-request) handles rate limiting and ephemeral caching. Resend + React Email manages transactional email (digests, job alerts). Meilisearch replaces Postgres FTS when typo tolerance and relevance tuning become user pain points — start with Postgres FTS, migrate when needed.

**Core technologies:**
- Next.js 15 + React 19: Full-stack framework — App Router and Server Components are the 2026 standard; reduces client bundle for SEO-sensitive pages
- PostgreSQL 16: Primary database — relational model maps directly to users/jobs/follows/comments; no document-DB impedance mismatch
- Drizzle ORM 0.38.x: Database access layer — serverless-first, no cold-start penalty, TypeScript schema definition
- Clerk: Authentication + user management — LinkedIn/Google OAuth, 10k MAU free, eliminates auth engineering overhead
- Tailwind CSS 4.x + shadcn/ui: Styling + component system — v4 drops config file; shadcn provides Card, Avatar, Badge, Feed components needed for this domain
- TanStack Query 5.x: Client server state — background refetch and optimistic updates for discussion threads
- Supabase Realtime 2.x: Live updates — uses Postgres logical replication; no separate WebSocket infrastructure
- Upstash Redis: Rate limiting + caching — serverless-native HTTP-based Redis; works on Vercel without persistent connections
- Resend + React Email: Transactional email — digest emails, job alerts, welcome flows; 3,000/month free
- Zod + React Hook Form: Validation — Server Action input validation + form management for profile creation and job posting flows
- Sentry 8.x: Error monitoring — official Next.js SDK with App Router and Server Action support

### Expected Features

Research identifies a clear three-tier structure: table stakes that make the platform feel real, differentiators that justify choosing CareerSignal over LinkedIn or Glassdoor, and deferred features that belong in v2 after product-market fit is established. The most important dependency: Community-Vetted Signal (the core differentiator) requires a critical mass of users to produce meaningful signal, which means it cannot be the first thing launched — the platform needs job listings and a seeded community first.

**Must have (table stakes):**
- User profile with career trajectory AND target roles — mid-career switchers need to show where they're going, not just where they've been
- Job listings (backfilled from external sources) — solves cold-start on supply side; gives users something to discuss from day one
- Company pages (basic, auto-generated from listings) — anchors company-level discussion; users need native context, not a link to Glassdoor
- Discussion threads on job listings — the core differentiator; this is what CareerSignal does that no competitor does
- Discussion threads on company pages — complements listing threads; users naturally move between the two
- Search and filter (role, location, industry, seniority) — users cannot engage with jobs they cannot find; filters must map to mid-career switcher mental models
- User auth (email + Google + LinkedIn OAuth) — LinkedIn OAuth is especially critical for professional network credibility
- Notifications for thread replies and followed company/role activity — closes the engagement loop; email digest default, not real-time
- Mobile-responsive design — non-negotiable from day one

**Should have (competitive):**
- Community-vetted signal (upvotes on listings, engagement activity scores) — the differentiator, but requires ~50+ active weekly users before signal is meaningful
- Salary transparency (crowdsourced, anonymous) — Levels.fyi proved this is one of the most powerful retention hooks in this space; suppress until 5+ submissions
- Saved job collections + personal notes — retention feature for repeat visitors tracking multiple opportunities
- Follow other professionals on career journey — asymmetric follow model (Twitter-style, not LinkedIn mutual); lowers anxiety of reaching out
- Career switcher success stories attached to listings — "someone went from teacher to UX designer here" is more valuable than any job description
- Employer/recruiter transparency scores (response rates) — needs minimum data volume and community trust before surfacing

**Defer (v2+):**
- Personalized job recommendations (ML-based) — requires data volume and engineering investment not justified pre-PMF; use rules-based target-role logic first
- "Is this role open to career changers?" formal signal — capture informally in discussion threads first, then formalize once patterns emerge
- Employer/recruiter portal (direct posting + community engagement tools) — brings monetization complexity that changes community dynamics before trust is built
- Native mobile app — web-first validates faster; React Native can consume the same Next.js API routes when the time comes

**Deliberate anti-features (never build these):**
- Cold DMs between strangers — creates LinkedIn-style recruiter spam that destroys community feel
- Full anonymity — Blind's experience shows it attracts harassment and bad-faith posts; use verified credentials with pseudonymous display instead
- Quick-apply to all listings — trains mass-apply behavior with 0% response rates; antithetical to community-informed job discovery
- Star ratings for employers — invites gaming and legal challenges; qualitative threads are richer and harder to game

### Architecture Approach

The architecture is a modular monolith in Next.js — all services (Feed, Jobs, Discussion, Auth/Profile, Notifications, Search) live within the same Next.js application, separated by module boundaries in code (`lib/actions/jobs.ts` vs `lib/actions/discussions.ts`) rather than by deployment. This is the correct choice at MVP: deployment complexity, distributed tracing overhead, and inter-service latency all outweigh scalability benefits until specific bottlenecks appear. The build sequence is architecturally determined by dependencies: Auth and Profiles first (every other feature requires identity), then Jobs and Companies (anchor entities), then Discussion (the core differentiator), then Follow Graph (enables personalization), then Feed (depends on follow graph + content), then Notifications (needs content loop working first), then Search (can be added incrementally).

The two most important data model decisions: (1) Job listings must own denormalized community signal (comment count, upvote count, last activity timestamp) from day one — feed queries should read one row per job card without expensive COUNT aggregates. (2) Comment threading is capped at 2 levels (top-level comment + one reply level) — deeper threading adds query complexity and UI noise without improving information quality in a job-focused context. Feed assembly uses a pull-based model at MVP (query on page load from followed entities, order by recency + activity score), which is simple to implement and sufficient through early growth, with a clear migration path to hybrid push/pull when feed query latency exceeds ~500ms p99.

**Major components:**
1. Auth + Profile Service — identity, session management, career trajectory data; Clerk handles OAuth, Postgres stores profile and career history
2. Job Service — CRUD for listings, enrichment with community signal scores; denormalized metadata on the Job entity enables fast feed rendering
3. Discussion Service — threaded comments (2-level max) anchored to jobs and companies; upvotes/reactions; adjacency-list tree in Postgres
4. Feed Service — pull-based assembly from follow graph + jobs + discussions; ordered by recency and community activity score
5. Notification Service — event-driven via Redis BullMQ queue; decoupled from write latency; email delivery via Resend; in-app via notifications table
6. Search Service — Postgres FTS at MVP; Typesense for production-grade faceted search with typo tolerance; search index synced via background worker

### Critical Pitfalls

1. **Cold start / ghost town** — Launch without seeding community discussion and every listing page looks like a plain job board. Prevention: recruit 20-50 founding members before public launch; every listing needs at least one community comment before surfacing broadly. Recovery cost is HIGH — this is the most common cause of early-stage community platform death.

2. **Audience drift destroys niche identity** — Adding new grad content, executive roles, or freelance gigs feels low-risk individually; collectively it makes mid-career switchers stop trusting the platform. Prevention: write the audience boundary ("mid-career professionals with 5+ years, switching industry or function — not new grads, not VP-level+") into product decisions before the first line of code. Enforce it in feed algorithm seniority filters.

3. **Employer side corrupts community trust** — Employer revenue is immediate; community trust erodes slowly and is nearly impossible to rebuild. Prevention: separate employer permissions from community content architecturally; employers can post listings, they cannot touch community commentary. Publish this as a public policy before any employer is onboarded.

4. **Stale listings destroy credibility** — One in five job listings in 2025 is a ghost job (no hiring intent). A platform with no freshness enforcement makes this worse. Prevention: 30-day automatic expiry with employer re-confirmation in the data model from day one; display "Last confirmed active" prominently on every listing.

5. **Community degrades to venting without signal** — Job search stress reliably turns semi-anonymous professional forums into venting boards. Prevention: design structured intel prompts at the discussion entry point from launch ("What's the interview process like?", "Is this role accurately described?"). Retrofitting structure onto an unstructured community is extremely hard.

## Implications for Roadmap

Based on the feature dependency graph, architecture build sequence, and pitfall-to-phase mapping from research, the following phase structure is recommended:

### Phase 1: Foundation (Auth, Profiles, and Listing Infrastructure)

**Rationale:** Everything else depends on identity (Auth/Profiles) and anchor entities (Jobs, Companies). The listing data model must include expiry/freshness logic from day one — this cannot be retrofitted. Audience boundary must be enforced at the data model and feed logic level before any user sees the platform.

**Delivers:** Working auth with Google + LinkedIn OAuth; user profiles with career trajectory AND target role fields; job listings with 30-day expiry rules and freshness signals; basic company pages auto-generated from listings; mobile-responsive design system.

**Addresses (from FEATURES.md):** User auth (P1), User profile (P1), Job listings backfilled (P1), Company pages basic (P1), Mobile-responsive design (P1).

**Avoids (from PITFALLS.md):** Stale listings (expiry in data model from day one); Audience drift (seniority/experience filters in listing schema); Cold start groundwork (listing infrastructure ready for seeding).

**Research flag:** STANDARD — well-documented Next.js + Clerk + Drizzle patterns; no deep phase research needed.

### Phase 2: Core Community Mechanic (Discussion on Listings and Companies)

**Rationale:** Discussion threads on listings are the core differentiator — the reason CareerSignal exists. This must be built before follow graphs, personalized feeds, or any other feature. The structured intel prompt format must be designed here, not added later. Founding member seeding happens during this phase before any public launch.

**Delivers:** Threaded discussion (2-level max) on job listings and company pages; structured intel prompt templates at comment entry point; upvote/reaction on comments; community signal (comment count, last activity) denormalized on Job entity; founding member community seeded (20-50 members with real commentary) before public launch.

**Addresses (from FEATURES.md):** Discussion threads on listings (P1), Discussion threads on companies (P1), Community-vetted signal groundwork.

**Avoids (from PITFALLS.md):** Cold start (seeding before public launch); Community degrades to venting (structured prompts at launch); Anti-patterns from ARCHITECTURE.md (shallow threading, denormalized signal on Job entity).

**Research flag:** NEEDS RESEARCH — discussion threading data model choices, structured intel prompt UX patterns, community seeding strategies for professional platforms.

### Phase 3: Discovery and Engagement (Search, Notifications, Follow Graph)

**Rationale:** Users cannot engage with jobs they cannot find. Search, notifications, and the follow graph close the engagement loop. These depend on jobs and discussion existing first (there needs to be something to search, something to be notified about, someone worth following). Notifications default to weekly digest to avoid notification fatigue.

**Delivers:** Search and filter for jobs (role, location, industry, seniority); in-app and email notifications for thread replies and followed entity activity (weekly digest default); follow graph for users and companies; basic personalized feed from follow graph (pull-based, ordered by recency + activity score).

**Addresses (from FEATURES.md):** Search and filter (P1), Notifications (P1), Follow other professionals (P2), Feed.

**Avoids (from PITFALLS.md):** Notification fatigue (daily digest default, frequency controls at signup); Recommendation cold start (show "popular in your target field" before behavioral data exists).

**Research flag:** STANDARD for notifications and follow graph (well-documented patterns). NEEDS RESEARCH for search UX — specifically how to structure faceted filters for career-change mental models (not just standard job board filters).

### Phase 4: Community Signal and Depth (Upvotes, Salary Transparency, Saved Jobs)

**Rationale:** These features require an active community to be credible. Salary data is misleading at n=1; upvote signal is meaningless without volume; saved jobs is a retention feature for repeat visitors. All three should launch after validating core engagement (rough threshold: 50+ active weekly users).

**Delivers:** Upvotes on job listings and comments; community activity scores surfaced on feed cards; crowdsourced salary data submissions (suppressed until 5+ submissions per company/role); saved job collections with personal notes; career switcher success stories attached to listings.

**Addresses (from FEATURES.md):** Community-vetted signal (P2), Salary transparency (P2), Saved job collections (P2), Career switcher success stories (P2).

**Avoids (from PITFALLS.md):** Premature salary display (5+ submission threshold enforced); Community signal requiring critical mass (gated on usage threshold before surfacing).

**Research flag:** NEEDS RESEARCH — salary transparency implementation models (Levels.fyi approach, verification mechanics, submission thresholds, display logic).

### Phase 5: Employer Side and Monetization

**Rationale:** Employer onboarding must happen after the community is large enough that losing one employer is tolerable. The community/employer boundary must be enforced architecturally and codified in public policy before any employer account is created. This is the highest-trust-risk phase in the platform's lifecycle.

**Delivers:** Employer account type with distinct permissions (listing creation only, no community moderation rights); employer badge on posts (visually distinct from peer intel); public community charter documenting employer/community separation; employer/recruiter transparency scores (response rates, surfaced after minimum data threshold); ATS webhook integration for automatic listing closure.

**Addresses (from FEATURES.md):** Employer response rate tracking (P2), ATS integrations.

**Avoids (from PITFALLS.md):** Employer/community trust corruption (architectural permission separation, public policy, no retroactive changes); Integration gotchas (ATS freshness timestamps required, ATS webhooks for listing status).

**Research flag:** NEEDS RESEARCH — employer permission architecture, community charter legal considerations, ATS webhook integration patterns (Greenhouse, Lever, Workday).

### Phase Ordering Rationale

- **Auth before everything:** The feature dependency graph has Auth as the root node — every community action requires identity. This is not debatable.
- **Jobs and Companies before Discussion:** Discussion has no anchor entity without job listings. The cold-start problem cannot be solved without listings to seed discussion onto.
- **Discussion before Feed:** The feed is populated by discussion activity. Building the feed before discussion produces an empty feed.
- **Community mass before Salary/Signal features:** Both crowdsourced salary data and upvote signals require minimum submission thresholds to be credible. Launching them before critical mass produces misleading data that damages trust.
- **Community trust before Employer onboarding:** Employer monetization introduces a trust risk that cannot be managed before the community is large enough to be resilient. Glassdoor and LinkedIn both eroded community trust by onboarding employers too early in their growth arc.
- **Monolith throughout:** Do not extract services until a specific bottleneck demands it. The first bottleneck will be feed query latency (add Redis caching); the second will be notification fanout under load.

### Research Flags

Phases likely needing deeper research during planning:
- **Phase 2 (Discussion):** Structured intel prompt UX is novel — limited prior art specific to job-anchored community discussion. Threading data model edge cases (comment edit history, threading on expired listings) need validation.
- **Phase 3 (Search):** Faceted search filters for career-change mental models are not standard job board patterns. Research needed on what filters mid-career switchers actually use vs. what generic job boards offer.
- **Phase 4 (Salary Transparency):** Levels.fyi's verification and suppression mechanics are not publicly documented. Research needed on submission verification approaches, display thresholds, and legal considerations around salary data.
- **Phase 5 (Employer Side):** ATS webhook integration patterns vary significantly by vendor (Greenhouse vs. Lever vs. Workday). Community charter legal language needs research.

Phases with standard patterns (skip research-phase):
- **Phase 1 (Foundation):** Next.js + Clerk + Drizzle + PostgreSQL is well-documented with official guides and high-confidence sources.
- **Phase 3 (Notifications + Follow Graph):** Event-driven notification with Redis BullMQ and asymmetric follow tables are standard patterns with extensive documentation.

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Core framework (Next.js 15, React 19) and tooling confirmed via official sources. Auth ecosystem shift (NextAuth → Better Auth/Clerk) confirmed via primary sources. One caveat: Drizzle v1.0 is in beta; stable 0.38.x recommended. |
| Features | MEDIUM | Based on competitor analysis and industry research (Levels.fyi, Glassdoor, Blind, LinkedIn). No direct user research data available. Feature priorities are informed inferences, not validated user research. |
| Architecture | MEDIUM | Feed, notification, and search patterns sourced from Stream.io and LinkedIn Engineering. Job portal data model sourced from practitioner literature. No official engineering blog posts from CareerSignal-equivalent products available. |
| Pitfalls | MEDIUM-HIGH | Cold start and audience drift patterns are well-documented across community platforms. Employer/community trust dynamics sourced from Glassdoor and LinkedIn case analysis. Platform-specific case studies limited by confidentiality. |

**Overall confidence:** MEDIUM-HIGH

### Gaps to Address

- **User research validation:** Feature priorities are inferred from competitor analysis, not direct user interviews with mid-career switchers. The first round of user research should validate: (a) which discussion prompts elicit actionable intel, (b) what filters mid-career switchers actually use for job discovery, (c) what salary transparency format is trusted vs. suspicious.
- **Listing supply strategy:** The research identifies the cold-start problem on listings (need content to anchor discussion) but does not resolve the best source: employer-direct postings, third-party aggregator APIs (Indeed/Adzuna/LinkedIn), or manual curation. This is a business and legal question, not just a technical one. Must be resolved before Phase 1 implementation.
- **Identity and verification model:** Pitfalls research recommends verified credentials with pseudonymous display. Architecture research notes work-email verification (Blind model). The specific verification flow (what qualifies as verification, how it's enforced, what happens to unverified users) is underdetermined and needs design research before Phase 2.
- **Monetization model:** Phase 5 (Employer side) is the implicit revenue model, but the research does not validate what employers would pay for or at what price point. Employer willingness to pay vs. community trust risk is the central business design question and deserves dedicated research before Phase 5 planning.
- **Auth.js / Better Auth ecosystem stability:** The merger between Auth.js and Better Auth was recent (as of research date). Better Auth is recommended for self-hosted scenarios, but the long-term stability and API compatibility of the merged project warrants monitoring before committing to it for GDPR/compliance variants of the stack.

## Sources

### Primary (HIGH confidence)
- [Next.js 15 release blog](https://nextjs.org/blog/next-15) — Stable status, React 19 compatibility, Turbopack
- [Auth.js joins Better Auth announcement](https://better-auth.com/blog/authjs-joins-better-auth) — Auth ecosystem shift confirmed
- [Clerk pricing page](https://clerk.com/pricing) — 10,000 MAU free tier confirmed
- [Supabase blog: Postgres FTS vs alternatives](https://supabase.com/blog/postgres-full-text-search-vs-the-rest) — FTS viability for early stage
- [shadcn/ui Tailwind v4 docs](https://ui.shadcn.com/docs/tailwind-v4) — Tailwind v4 compatibility confirmed
- [TanStack Query v5 SSR guide](https://tanstack.com/query/v5/docs/react/guides/advanced-ssr) — RSC + streaming support
- [Stream.io: Scalable Activity Feed Architecture](https://getstream.io/blog/scalable-activity-feed-architecture/) — Push/pull/hybrid feed patterns
- [Modern Full-Stack Next.js 15 Architecture](https://softwaremill.com/modern-full-stack-application-architecture-using-next-js-15/) — Server/Client component boundaries

### Secondary (MEDIUM confidence)
- [Levels.fyi](https://www.levels.fyi/) — Salary transparency model analysis
- [How Levels.fyi Brought Salary Transparency to Tech](https://www.tryexponent.com/blog/levels-interview-tech-salary-transparency) — Crowdsourced salary mechanics
- [Career Switchers: Most Valuable Talent Pool in 2026](https://idealtraits.com/blog/career-switchers-the-most-valuable-talent-pool-in-2026/) — Mid-career switcher market sizing
- [LinkedIn System Design Handbook](https://www.systemdesignhandbook.com/guides/linkedin-system-design-interview/) — Feed and notification architecture patterns
- [Ghost Jobs: From 1 in 8 to 1 in 5](https://medium.com/@patricklindsley1/from-1-in-8-to-1-in-5-ghost-jobs-are-exploding-in-2025-a7a1fbdad011) — Stale listing problem severity
- [The Cold Start Problem](https://read.thecoder.cafe/p/cold-start-problem) — Network effect bootstrapping patterns
- [Coding Horror: Web Discussions Flat by Design](https://blog.codinghorror.com/web-discussions-flat-by-design/) — Rationale for shallow threading
- [Cracking the chicken and egg problem for job boards](https://www.smartjobboard.com/blog/chicken-egg-problem-for-job-boards/) — Cold-start strategy for two-sided job platforms
- [State of the Job Search 2025 — Jobscan](https://www.jobscan.co/state-of-the-job-search) — Quick-apply failure rates

### Tertiary (LOW confidence — needs validation during implementation)
- [Typesense vs Algolia vs Elasticsearch](https://typesense.org/typesense-vs-algolia-vs-elasticsearch-vs-meilisearch/) — Search infrastructure comparison (official Typesense source; treat as biased)
- [LinkedIn Concourse Notification Platform](https://engineering.linkedin.com/blog/2018/05/concourse--generating-personalized-content-notifications-in-near) — Notification routing patterns (2018; architecture may be outdated)
- [Glassdoor anonymous community features](https://techcrunch.com/2023/07/18/glassdoor-is-introducing-blind-like-anonymous-community-features-to-fuel-user-growth/) — Competitor feature analysis (journalistic, not engineering)

---
*Research completed: 2026-03-04*
*Ready for roadmap: yes*
