# Architecture Research

**Domain:** Community-driven professional job discovery platform
**Researched:** 2026-03-04
**Confidence:** MEDIUM (web sources + system design literature; no official platform engineering blog posts from CareerSignal-equivalent products)

## Standard Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                           Client Layer                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Job Feed   │  │  Job Detail  │  │   Profile    │              │
│  │    Page      │  │  + Comments  │  │    Page      │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                  │                      │
└─────────┼─────────────────┼──────────────────┼──────────────────────┘
          │                 │                  │
┌─────────▼─────────────────▼──────────────────▼──────────────────────┐
│                      Application Layer (Next.js)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │  Feed        │  │  Discussion  │  │   Auth &     │              │
│  │  Service     │  │  Service     │  │   Profile    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                  │                      │
│  ┌──────▼───────┐  ┌──────▼───────┐  ┌──────▼───────┐              │
│  │  Job         │  │  Notif.      │  │   Search     │              │
│  │  Service     │  │  Service     │  │   Service    │              │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘              │
│         │                 │                  │                      │
└─────────┼─────────────────┼──────────────────┼──────────────────────┘
          │                 │                  │
┌─────────▼─────────────────▼──────────────────▼──────────────────────┐
│                         Data Layer                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │  PostgreSQL  │  │    Redis     │  │  Search      │              │
│  │  (primary)   │  │  (cache +    │  │  Index       │              │
│  │              │  │   queues)    │  │  (Typesense) │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| Component | Responsibility | Typical Implementation |
|-----------|----------------|------------------------|
| Feed Service | Assembles personalized job + discussion activity for each user | Pull-based at MVP; hybrid push/pull at scale |
| Job Service | CRUD for job listings; enrichment with community signal scores | PostgreSQL + denormalized community metadata |
| Discussion Service | Threaded comments anchored to jobs and companies; upvotes/reactions | Adjacency-list tree in Postgres; depth capped at 2 |
| Auth + Profile Service | Identity, session management, career trajectory data | Auth.js / NextAuth; profile stored in Postgres |
| Notification Service | Delivers in-app, email alerts on replies, follows, job updates | Event queue (Redis BullMQ or Kafka at scale) + delivery adapters |
| Search Service | Full-text search across jobs, companies, profiles, discussions | Typesense (self-hosted) or Algolia for managed |
| Follow Graph | Tracks user–user and user–company follow relationships | Simple adjacency table in Postgres at MVP; dedicated graph store at scale |

## Recommended Project Structure

```
src/
├── app/                     # Next.js App Router (pages + API routes)
│   ├── (auth)/              # Authentication routes (login, signup, onboarding)
│   ├── (feed)/              # Home feed, following feed
│   ├── jobs/                # Job listing detail + community thread
│   ├── companies/           # Company profile + all discussions
│   ├── profile/             # User profile pages
│   └── api/                 # API routes not covered by Server Actions
│
├── components/              # Shared UI components
│   ├── feed/                # FeedCard, FeedList, FeedFilter
│   ├── jobs/                # JobCard, JobDetail, JobBadge
│   ├── discussion/          # CommentTree, CommentInput, ReactionBar
│   ├── profile/             # ProfileCard, CareerTimeline
│   └── notifications/       # NotificationBell, NotificationList
│
├── lib/                     # Business logic, services, data access
│   ├── actions/             # Server Actions (mutations)
│   │   ├── jobs.ts
│   │   ├── discussions.ts
│   │   ├── profiles.ts
│   │   └── follows.ts
│   ├── queries/             # Read-only data fetching
│   │   ├── feed.ts
│   │   ├── search.ts
│   │   └── notifications.ts
│   ├── db/                  # Prisma client, schema, migrations
│   └── search/              # Search index sync, query builders
│
├── workers/                 # Background jobs (notification dispatch, feed cache)
│   ├── notification.worker.ts
│   └── search-sync.worker.ts
│
└── types/                   # Shared TypeScript types
```

### Structure Rationale

- **app/ (App Router):** Co-locates page rendering with data fetching via Server Components. Job detail pages fetch job data + first page of comments server-side for fast initial load.
- **lib/actions/ vs lib/queries/:** Explicit separation of write and read paths prevents accidental mutation in read contexts and makes caching safe (queries can be cached; actions cannot).
- **workers/:** Background processing (notification delivery, search index updates) runs outside the request cycle. At MVP this can be a simple Redis queue; it doesn't need a separate process until traffic demands it.

## Architectural Patterns

### Pattern 1: Job as Conversation Anchor

**What:** Every job listing is a first-class entity that community discussion threads attach to. Jobs are not just data records — they are the canonical context for all user-generated content. The job record stores both the raw posting data and denormalized community signal (comment count, upvote count, last activity timestamp).

**When to use:** Always — this is the core data model for CareerSignal. Community signal is what differentiates listings from a plain job board.

**Trade-offs:** Denormalized counters require careful update logic (increment-on-write, reconcile async); avoids expensive COUNT queries on every feed render.

**Example:**
```typescript
// lib/db/schema.ts (Prisma)
model Job {
  id                String    @id @default(cuid())
  title             String
  company           Company   @relation(fields: [companyId], references: [id])
  companyId         String
  description       String
  postedAt          DateTime
  // Denormalized community signal — updated by trigger or worker
  commentCount      Int       @default(0)
  upvoteCount       Int       @default(0)
  lastActivityAt    DateTime  @default(now())
  comments          Comment[]
  upvotes           JobUpvote[]
}
```

### Pattern 2: Pull-Based Feed at MVP (Switch to Hybrid When Needed)

**What:** On feed page load, query jobs and discussions from accounts the user follows, rank by recency + community activity score. At MVP scale (< 10k users), on-demand query is fast enough and avoids fan-out write complexity.

**When to use:** MVP through early growth. Migrate to hybrid push/pull once feed queries exceed ~500ms p99 or background job infrastructure is in place.

**Trade-offs:** Simple to implement and reason about. Becomes slow at scale when a user follows thousands of accounts or a popular job generates thousands of comments. Pure pull makes ML ranking harder (must rank at read time).

**Example:**
```typescript
// lib/queries/feed.ts
export async function getFeedForUser(userId: string, cursor?: string) {
  // Get IDs of users + companies this user follows
  const following = await db.follow.findMany({ where: { followerId: userId } });
  const followedUserIds = following.filter(f => f.targetType === 'USER').map(f => f.targetId);
  const followedCompanyIds = following.filter(f => f.targetType === 'COMPANY').map(f => f.targetId);

  // Pull jobs with community activity from followed sources
  return db.job.findMany({
    where: {
      OR: [
        { companyId: { in: followedCompanyIds } },
        { comments: { some: { authorId: { in: followedUserIds } } } }
      ]
    },
    orderBy: { lastActivityAt: 'desc' },
    take: 20,
    cursor: cursor ? { id: cursor } : undefined,
    include: { company: true, _count: { select: { comments: true } } }
  });
}
```

### Pattern 3: Shallow Comment Trees (Max 2 Levels)

**What:** Discussion threads on jobs are flat or 2-level deep (top-level comment + one reply level). This matches how platforms like Hacker News and Product Hunt structure discussions — deep threading adds visual complexity without improving conversation quality on job-focused content.

**When to use:** Always for job-anchored discussion. Deeper threads can be considered for company "culture" discussion boards later.

**Trade-offs:** Simpler data model (no recursive queries), cleaner UI, faster to build. Loses nested conversational depth that Reddit-style platforms offer — acceptable tradeoff for a job discovery context where brevity is valued.

**Example:**
```typescript
model Comment {
  id         String    @id @default(cuid())
  body       String
  author     User      @relation(fields: [authorId], references: [id])
  authorId   String
  job        Job       @relation(fields: [jobId], references: [id])
  jobId      String
  parent     Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  parentId   String?   // null = top-level; non-null = reply (max 1 level)
  replies    Comment[] @relation("CommentReplies")
  createdAt  DateTime  @default(now())
  @@index([jobId, createdAt])
}
```

## Data Flow

### Request Flow — Job Detail Page Load

```
User navigates to /jobs/[id]
    |
    v
Next.js Server Component (app/jobs/[id]/page.tsx)
    |
    +--> lib/queries/jobs.ts → PostgreSQL (job + company data)
    |
    +--> lib/queries/discussions.ts → PostgreSQL (top 20 comments + reply counts)
    |
    v
HTML streamed to browser (fast initial render, no JS required)
    |
    v
Client hydrates → CommentInput component enables posting
```

### Request Flow — Post a Comment (Server Action)

```
User submits CommentInput
    |
    v
Server Action: lib/actions/discussions.ts:createComment()
    |
    +--> Write comment to PostgreSQL
    |
    +--> Increment job.commentCount (denormalized)
    |
    +--> Update job.lastActivityAt
    |
    +--> Enqueue notification event → Redis BullMQ
    |         |
    |         v
    |     notification.worker.ts consumes event
    |         +--> Lookup job author + thread subscribers
    |         +--> Send in-app notification (write to notifications table)
    |         +--> Send email (via Resend / SendGrid) if user opted in
    |
    v
Revalidate feed cache (Next.js revalidatePath or revalidateTag)
```

### Request Flow — Search

```
User types in search bar
    |
    v
Client-side debounce (300ms)
    |
    v
API route: /api/search?q=[query]&type=[jobs|companies|people]
    |
    v
Search Service → Typesense HTTP API
    |
    v
Results returned + hydrated with Postgres data (company logo, etc.)
    |
    v
Search results rendered client-side (instant feel)

Background: Postgres writes trigger search index sync via worker
(job created → worker pushes to Typesense within ~1s)
```

### Key Data Flows Summary

1. **Job enrichment:** Job listings flow from external postings (employer-created) into Postgres. Community activity (comments, upvotes) flows back and decorates each job record with signal scores.
2. **Feed assembly:** User's follow graph (who/what they follow) is the input. Recent job activity from followed entities is the output. All assembled at query time at MVP.
3. **Notification fanout:** A user action (comment, follow, job post) writes to Postgres, emits an event to a queue, and the worker resolves who to notify and via what channel. The action is decoupled from delivery.
4. **Search sync:** Writes to Postgres are the source of truth. A background sync pushes changes to the search index. Search queries hit Typesense (fast), not Postgres (slow for full-text).

## Scaling Considerations

| Scale | Architecture Adjustments |
|-------|--------------------------|
| 0–1k users | Full monolith. All services in Next.js API routes and Server Actions. Single Postgres instance. Redis for queues. No separate workers (use Vercel Cron or similar). Typesense on a small VPS. |
| 1k–100k users | Add Redis caching layer for feed queries and user session data. Separate notification worker process. Add read replica for Postgres. Enable CDN caching on company/job static data. |
| 100k+ users | Hybrid push/pull feed (materialize feeds for active users). Dedicated search infrastructure. Consider extracting notification and search as independent services. Sharding follow graph. |

### Scaling Priorities

1. **First bottleneck — feed query latency:** Pull-based feed queries join across follows + jobs + comments. This degrades first. Fix: add Redis caching with short TTL (60s), then switch to hybrid feed materialization.
2. **Second bottleneck — notification fanout:** When a popular job attracts hundreds of comments, the notification queue depth grows. Fix: priority queues (replies to you = high, weekly digest = low), batch email delivery, rate limit notifications per user per day.

## Anti-Patterns

### Anti-Pattern 1: Treating Jobs as Static Listings

**What people do:** Model job listings as plain CRUD records, add comments as an afterthought in a separate module with a foreign key.

**Why it's wrong:** The whole product value is community signal on jobs. If the job entity doesn't own its activity metadata (comment count, last activity, community sentiment), every feed render requires expensive joins. Community context becomes a second-class citizen in the data model and the UI.

**Do this instead:** Design the Job entity to own denormalized community signal from day one. Feed queries read one row per job with all signal needed to render a card.

### Anti-Pattern 2: Deep Comment Threading

**What people do:** Implement Reddit-style infinite nesting (comments → replies → replies → replies) because it feels complete.

**Why it's wrong:** Job-focused discussion doesn't benefit from deep threads. It creates complex recursive queries, harder UI, and tends to bury the most useful top-level intel (salary, culture, red flags) in thread depth. Most job platforms that attempted deep threading (Glassdoor, early Blind) heavily moderate and surface top-level comments anyway.

**Do this instead:** Cap at 2 levels. Top-level comment = "here's what I know about this role." Reply = "thanks / follow-up question." This maps to how mid-career switchers actually consume information.

### Anti-Pattern 3: Building Microservices Before Product-Market Fit

**What people do:** Separate auth, jobs, discussions, notifications, search, and feed into independent deployed services from day one.

**Why it's wrong:** At early stage, deployment complexity, distributed tracing, and inter-service latency costs far outweigh any scalability benefit. The product will need to iterate rapidly. Coupling through a shared database is fine and expected at this scale.

**Do this instead:** Start with a modular monolith in Next.js. Separate services by module boundaries in code (lib/actions/jobs.ts vs lib/actions/discussions.ts), not deployment. Extract to services only when a specific bottleneck demands it.

### Anti-Pattern 4: Embedding Search in the Primary Database

**What people do:** Use Postgres full-text search (tsvector/tsquery) for job and profile search.

**Why it's wrong:** Postgres full-text works initially but degrades under: typo tolerance requirements, faceted filtering (role + location + company size), relevance tuning, and instant-search UX. These are table stakes for a job discovery platform.

**Do this instead:** Stand up Typesense from day one. It's a single binary, trivial to self-host, and adds typo tolerance and fast faceted search immediately. Keep Postgres as source of truth; sync to Typesense on writes.

## Integration Points

### External Services

| Service | Integration Pattern | Notes |
|---------|---------------------|-------|
| Auth provider (GitHub/Google) | OAuth via Auth.js / NextAuth; tokens stored in Postgres sessions table | Use provider OAuth at MVP to avoid password management; add email/password later |
| Email delivery (Resend / SendGrid) | HTTP API call from notification worker | Don't send email synchronously in request cycle — always enqueue |
| Job aggregators (optional: LinkedIn API, Indeed, Adzuna) | Scheduled ingestion job → normalize into Job schema | External job feeds are supplemental; employer-posted + community-submitted jobs are primary |
| Typesense | HTTP API from search service module; sync writes via worker | Self-host on a $10/mo VPS; avoid Algolia until scale demands it |

### Internal Boundaries

| Boundary | Communication | Notes |
|----------|---------------|-------|
| Feed Service ↔ Job Service | Direct Prisma queries (same DB) | Feed queries denormalized job fields; no cross-service HTTP at MVP |
| Discussion Service ↔ Notification Service | Redis BullMQ queue (event-driven) | Comment creation emits event; notification worker subscribes. Decouples write latency from notification delivery. |
| Job Service ↔ Search Service | Worker polls for new/updated jobs and pushes to Typesense | Eventually consistent; search index lags Postgres by < 2s |
| Auth ↔ All Services | Session token in HTTP cookie; validated via Auth.js middleware | All server actions check session before mutating. Centralize auth check in a single middleware wrapper. |

## Build Order Implications

The component dependencies create a natural build sequence:

1. **Auth + Profiles** — Everything requires identity. Build this first. Users can't follow, post, or be notified without an account.
2. **Job Listings (read + create)** — The anchor entity. Build job posting and display before community features attach to jobs.
3. **Company Profiles** — Companies are the parent context for jobs. Build alongside or immediately after jobs.
4. **Discussion (comments on jobs)** — Depends on jobs existing. This is the core differentiator — prioritize over follows/feed.
5. **Follow Graph** — Depends on profiles. Enables personalized feed but not required for basic discussion.
6. **Feed** — Depends on follow graph + jobs + discussions. Pull-based query is straightforward once the underlying data exists.
7. **Notifications** — Depends on discussion + follow graph. Add after the content loop is working so there's something to notify about.
8. **Search** — Can be added incrementally. Typesense sync can start after jobs + profiles exist. Full faceted search is a refinement, not a blocker.

## Sources

- [LinkedIn System Design Interview Guide](https://www.systemdesignhandbook.com/guides/linkedin-system-design-interview/) — Feed architecture, notification patterns, event-driven design (MEDIUM confidence — system design guide, not official LinkedIn engineering post)
- [LinkedIn Concourse Notification Platform](https://engineering.linkedin.com/blog/2018/05/concourse--generating-personalized-content-notifications-in-near) — Recipient-based notification routing (MEDIUM confidence — official but 2018)
- [Stream.io: Scalable Activity Feed Architecture](https://getstream.io/blog/scalable-activity-feed-architecture/) — Push/pull/hybrid feed patterns (HIGH confidence — authoritative practitioner source)
- [Fanout at Scale: Push vs Pull](https://dev.to/ahsanfarooq210/fanout-at-scale-push-vs-pull-strategies-in-distributed-systems-228l) — Hybrid approach rationale (MEDIUM confidence)
- [Designing a Database for an Online Job Portal](https://www.red-gate.com/blog/designing-a-database-for-an-online-job-portal/) — Core job portal schema entities (MEDIUM confidence)
- [Modern Full-Stack Next.js 15 Architecture](https://softwaremill.com/modern-full-stack-application-architecture-using-next-js-15/) — Server/Client component boundaries (HIGH confidence — current, official framework patterns)
- [Typesense vs Algolia vs Elasticsearch Comparison](https://typesense.org/typesense-vs-algolia-vs-elasticsearch-vs-meilisearch/) — Search infrastructure choice (HIGH confidence — official Typesense documentation)
- [System Design: Comment Reply System](https://dilipkumar.medium.com/comment-reply-system-design-6987b9971809) — Threaded comment data models (MEDIUM confidence)
- [Coding Horror: Web Discussions Flat by Design](https://blog.codinghorror.com/web-discussions-flat-by-design/) — Rationale for shallow threading (MEDIUM confidence — practitioner opinion, well-regarded source)

---
*Architecture research for: CareerSignal — community-driven professional job discovery*
*Researched: 2026-03-04*
