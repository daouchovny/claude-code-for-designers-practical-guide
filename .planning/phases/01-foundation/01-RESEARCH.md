# Phase 1: Foundation - Research

**Researched:** 2026-03-04
**Domain:** Next.js 15 + Clerk auth + Drizzle ORM + PostgreSQL + job listing data model
**Confidence:** HIGH (standard, well-documented stack with official sources)

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| AUTH-01 | User can create an account with email and password | Clerk's `<SignUp />` component handles email/password out of the box with no custom auth code |
| AUTH-02 | User can sign in with Google OAuth | Clerk enables Google OAuth via dashboard toggle; zero code changes required |
| AUTH-03 | User can sign in with LinkedIn OAuth | Clerk enables LinkedIn OAuth via dashboard toggle; same pattern as Google |
| AUTH-04 | User session persists across browser refresh | Clerk manages session tokens in cookies automatically; no custom session logic needed |
| AUTH-05 | User can sign out from any page | Clerk's `<UserButton />` component includes sign-out; or call `signOut()` from any component |
| JOBS-01 | User can browse a paginated list of job listings | Drizzle `.limit()` + `.offset()` or cursor-based pagination on the `jobs` table; rendered as RSC |
| JOBS-02 | User can view a job listing detail page | `/jobs/[id]` page.tsx as a Server Component; single Drizzle query joins job + company |
| JOBS-03 | User can save a job listing to their saved jobs | `saved_jobs` join table (user_id, job_id, saved_at); Server Action toggles the row |
| JOBS-04 | User can view and manage their saved jobs list | `/profile/saved` page queries `saved_jobs` joined to `jobs`; delete via Server Action |
| JOBS-05 | Job listings display posted date and are marked stale after 30 days | `posted_at` column on `jobs`; stale = `posted_at < NOW() - INTERVAL '30 days'`; computed at query or display time |
</phase_requirements>

---

## Summary

Phase 1 covers two co-dependent domains: authentication (identity infrastructure) and job listing inventory (anchor content). These are the root dependencies for every other phase — community discussion has no anchor without listings, and listings have no ownership or personalization without identity. The research confirms this phase uses well-documented, stable patterns with high confidence.

Clerk is the correct auth choice for this platform. It handles email/password, Google OAuth, and LinkedIn OAuth through a dashboard configuration — no custom OAuth flows required. Session persistence and sign-out are built-in. The alternative (Better Auth) is worth knowing about but only relevant if GDPR data residency requirements emerge.

The job listing data model is the most consequential decision in Phase 1. The model must carry denormalized community signal columns (`comment_count`, `last_activity_at`) from the first migration — retrofitting these later creates expensive backfills and query rewrites. The 30-day staleness rule belongs in the schema as a computed query predicate, not in application logic.

**Primary recommendation:** Scaffold with `create-next-app`, configure Clerk middleware, define the Drizzle schema with community signal columns pre-built, seed 5-10 job listings manually, and ship the paginated listing page and save functionality before adding any UI polish.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Next.js | 15.x | Full-stack framework | App Router + Server Components are the 2026 standard; RSC handles SEO-critical job pages without client JS |
| React | 19.x | UI layer | Ships with Next.js 15; stable with App Router |
| TypeScript | 5.x | Type safety | Required for complex interconnected schemas (jobs, users, saved_jobs) |
| PostgreSQL | 16.x | Primary database | Relational model maps directly to this domain; no document-DB impedance mismatch |
| Drizzle ORM | 0.38.x | Database access | Serverless-first (no Rust engine cold-start on Vercel); TypeScript schema definition; ~7kb bundle |
| Clerk | latest | Auth + session management | Handles email/password + Google + LinkedIn OAuth; 10k MAU free; SOC 2 Type II; eliminates weeks of auth engineering |
| Tailwind CSS | 4.x | Styling | 2026 standard for Next.js; v4 drops config file in favor of CSS-native `@theme` directives |
| shadcn/ui | latest | Component library | Owned source (not a package); provides Card, Badge, Avatar, Skeleton needed for job cards and listing pages |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| Zod | 3.x | Schema validation | Validate Server Action inputs (job saves, pagination params) |
| React Hook Form | 7.x | Form management | Any form that needs client-side validation before Server Action submission |
| Drizzle Kit | latest | Migrations CLI | `drizzle-kit push` for dev; `drizzle-kit migrate` for production deployments |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Clerk | Better Auth | Only if GDPR data residency requires self-hosted auth; adds operational overhead |
| Clerk | Supabase Auth | Only if committing to full Supabase stack; less polished for LinkedIn OAuth |
| Drizzle ORM | Prisma 7.x | Prisma 7 closed the serverless performance gap; still larger bundle than Drizzle |
| Next.js Server Actions | tRPC | tRPC adds value in monorepos with separate frontend/backend teams; no benefit in this project |

**Installation:**
```bash
# Bootstrap
npx create-next-app@latest careersignal --typescript --tailwind --eslint --app --src-dir

# Database + ORM
npm install drizzle-orm postgres
npm install -D drizzle-kit

# Auth
npm install @clerk/nextjs

# Validation
npm install zod react-hook-form @hookform/resolvers

# UI components (copy source, not package install)
npx shadcn@latest init
npx shadcn@latest add button card badge avatar skeleton separator
```

---

## Architecture Patterns

### Recommended Project Structure

```
src/
├── app/
│   ├── (auth)/              # Clerk sign-in, sign-up, onboarding routes
│   ├── jobs/
│   │   ├── page.tsx         # Paginated job listing (RSC)
│   │   └── [id]/
│   │       └── page.tsx     # Job detail page (RSC)
│   ├── profile/
│   │   └── saved/
│   │       └── page.tsx     # Saved jobs list (RSC)
│   └── api/                 # Minimal — prefer Server Actions over API routes
│
├── components/
│   ├── jobs/                # JobCard, JobDetail, StaleBadge, SaveButton
│   └── ui/                  # shadcn-owned components (Button, Card, Badge, etc.)
│
├── lib/
│   ├── db/
│   │   ├── schema.ts        # Drizzle schema — source of truth
│   │   └── index.ts         # db client singleton
│   ├── actions/
│   │   └── jobs.ts          # saveJob(), unsaveJob() Server Actions
│   └── queries/
│       ├── jobs.ts          # getJobs(), getJobById(), getSavedJobs()
│       └── users.ts         # getUserProfile()
│
└── types/
    └── index.ts             # Shared TypeScript types
```

### Pattern 1: Clerk Middleware for Route Protection

**What:** A single `middleware.ts` at the project root uses Clerk's `clerkMiddleware()` to protect authenticated routes. Public routes (job listing, job detail) are explicitly declared; everything else requires auth.

**When to use:** Always — set this up before writing any page component.

```typescript
// middleware.ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isPublicRoute = createRouteMatcher([
  '/',
  '/jobs(.*)',           // job listings and detail pages are public
  '/sign-in(.*)',
  '/sign-up(.*)',
])

export default clerkMiddleware(async (auth, request) => {
  if (!isPublicRoute(request)) {
    await auth.protect()
  }
})

export const config = {
  matcher: ['/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jinja2|txt|xml|ico|webp|png|jpg|gif|svg|ttf|woff2?|eot|otf|wasm)).*)', '/(api|trpc)(.*)'],
}
```

### Pattern 2: Drizzle Schema with Community Signal Pre-Built

**What:** The `jobs` table carries `comment_count` and `last_activity_at` denormalized columns from migration 0001. These are updated by Server Actions when comments are posted (Phase 2), but the columns must exist in Phase 1 to avoid a schema migration mid-phase-2.

**When to use:** Always — define in the initial schema, default to 0 / now().

```typescript
// lib/db/schema.ts
import { pgTable, text, integer, timestamp, boolean, uuid } from 'drizzle-orm/pg-core'

export const jobs = pgTable('jobs', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: text('title').notNull(),
  company: text('company').notNull(),
  description: text('description').notNull(),
  location: text('location').notNull(),
  roleType: text('role_type').notNull(), // 'full-time' | 'contract' | 'part-time'
  postedAt: timestamp('posted_at').notNull().defaultNow(),
  expiresAt: timestamp('expires_at'),    // set to postedAt + 30 days on insert
  // Community signal — pre-built in Phase 1, populated in Phase 2
  commentCount: integer('comment_count').notNull().default(0),
  lastActivityAt: timestamp('last_activity_at').notNull().defaultNow(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
})

export const savedJobs = pgTable('saved_jobs', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: text('user_id').notNull(), // Clerk user ID (string, not UUID)
  jobId: uuid('job_id').notNull().references(() => jobs.id, { onDelete: 'cascade' }),
  savedAt: timestamp('saved_at').notNull().defaultNow(),
})
```

### Pattern 3: Staleness as a Query Predicate, Not Application Logic

**What:** A job is "stale" if `posted_at < NOW() - INTERVAL '30 days'`. This is computed in the database query (or derived at render time from the `posted_at` value), not stored as a boolean flag that requires a cron job to update.

**When to use:** For JOBS-05 display; for future listing expiry enforcement.

```typescript
// lib/queries/jobs.ts
import { db } from '@/lib/db'
import { jobs } from '@/lib/db/schema'
import { desc, sql } from 'drizzle-orm'

export async function getJobs({ page = 1, limit = 20 }: { page?: number; limit?: number }) {
  const offset = (page - 1) * limit
  return db
    .select({
      id: jobs.id,
      title: jobs.title,
      company: jobs.company,
      location: jobs.location,
      roleType: jobs.roleType,
      postedAt: jobs.postedAt,
      commentCount: jobs.commentCount,
      // Staleness computed at query time — no cron job needed
      isStale: sql<boolean>`${jobs.postedAt} < NOW() - INTERVAL '30 days'`,
    })
    .from(jobs)
    .orderBy(desc(jobs.lastActivityAt))
    .limit(limit)
    .offset(offset)
}
```

### Pattern 4: Save/Unsave via Server Action with Optimistic UI

**What:** Saving a job writes a row to `saved_jobs`. The Server Action validates the Clerk session, upserts the record, and revalidates the saved jobs path.

**When to use:** JOBS-03 and JOBS-04.

```typescript
// lib/actions/jobs.ts
'use server'
import { auth } from '@clerk/nextjs/server'
import { revalidatePath } from 'next/cache'
import { db } from '@/lib/db'
import { savedJobs } from '@/lib/db/schema'
import { eq, and } from 'drizzle-orm'

export async function saveJob(jobId: string) {
  const { userId } = await auth()
  if (!userId) throw new Error('Unauthorized')

  await db.insert(savedJobs).values({ userId, jobId }).onConflictDoNothing()
  revalidatePath('/profile/saved')
}

export async function unsaveJob(jobId: string) {
  const { userId } = await auth()
  if (!userId) throw new Error('Unauthorized')

  await db.delete(savedJobs).where(
    and(eq(savedJobs.userId, userId), eq(savedJobs.jobId, jobId))
  )
  revalidatePath('/profile/saved')
}
```

### Pattern 5: Job Detail Page as Server Component

**What:** `/jobs/[id]/page.tsx` is a Server Component that fetches job data server-side. No client-side fetching for the initial render — fast, SEO-friendly, no loading spinner.

```typescript
// app/jobs/[id]/page.tsx
import { db } from '@/lib/db'
import { jobs } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'
import { notFound } from 'next/navigation'

export default async function JobDetailPage({ params }: { params: { id: string } }) {
  const job = await db.select().from(jobs).where(eq(jobs.id, params.id)).limit(1)
  if (!job.length) notFound()
  // Render job data directly — no useEffect, no loading state
  return <JobDetail job={job[0]} />
}
```

### Anti-Patterns to Avoid

- **Using `useEffect` to fetch job data on the client:** Server Components fetch data at render time; no client fetch needed for initial page load.
- **Storing `isStale` as a database column with a cron updater:** Staleness is a time calculation, not a stored state. Compute it in the query.
- **Rolling custom session management:** Clerk handles this. Do not write middleware that reads JWT cookies manually.
- **Putting Drizzle client instantiation inside Server Actions:** Create a singleton `db` client in `lib/db/index.ts` and import it everywhere.
- **Using NextAuth.js v4 or Auth.js v5 beta:** Both are deprecated/unstable for new projects as of 2026. Use Clerk.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Email/password auth + session cookies | Custom JWT + bcrypt + cookie middleware | Clerk | Session rotation, CSRF, token refresh — all edge-case-heavy; Clerk handles it correctly |
| OAuth flow (Google, LinkedIn) | Custom OAuth callback routes + token exchange | Clerk dashboard toggle | OAuth flows have dozens of failure modes; Clerk's pre-built components cover them all |
| Pagination UI | Custom cursor/offset state machine | Drizzle `.limit()` + `.offset()` + Next.js search params | Off-by-one errors, race conditions on rapid navigation |
| Stale listing detection | Cron job that sets `is_stale = true` | SQL predicate: `posted_at < NOW() - INTERVAL '30 days'` | Cron jobs drift; SQL predicates are always accurate |
| Form validation | Manual regex + error state | Zod + React Hook Form | Async validation, nested object validation, and error message coordination are handled |

**Key insight:** Auth and form validation are among the most error-prone domains in web development. Both have well-tested, purpose-built libraries for the Next.js stack. Any hand-rolled replacement will have security gaps or UX regressions within the first month.

---

## Common Pitfalls

### Pitfall 1: Missing Clerk Environment Variables

**What goes wrong:** `clerkMiddleware()` silently fails or throws an unreadable error if `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` or `CLERK_SECRET_KEY` are missing.

**Why it happens:** Clerk requires both variables; forgetting one in `.env.local` or Vercel environment config breaks all auth routes.

**How to avoid:** Add a startup check. Clerk's Next.js SDK will throw a clear error on boot if the keys are absent — verify both keys are set before deploying.

**Warning signs:** `/sign-in` returns a 500; Clerk components render blank; `auth()` returns `null` even for signed-in users.

### Pitfall 2: Forgetting `'use server'` on Server Actions

**What goes wrong:** A Server Action without `'use server'` at the top of the file (or function) runs on the client, exposing database credentials and breaking Drizzle imports.

**Why it happens:** Next.js requires the directive explicitly; there is no automatic detection.

**How to avoid:** Every file in `lib/actions/` should start with `'use server'`. Lint rule: `next/no-missing-server-directive` catches this.

### Pitfall 3: Drizzle Client Instantiated Per-Request

**What goes wrong:** Creating a new `drizzle(postgres(...))` instance inside a Server Action or page component creates a new connection pool per request. On Vercel serverless, this exhausts PostgreSQL connection limits rapidly.

**Why it happens:** Developers copy-paste the Drizzle quickstart which shows inline instantiation.

**How to avoid:** Export a singleton from `lib/db/index.ts`. Use the global object pattern for Vercel's hot-reload environment:

```typescript
// lib/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'

const client = postgres(process.env.DATABASE_URL!)
export const db = drizzle(client)
```

### Pitfall 4: Clerk User ID Type Mismatch with UUID Columns

**What goes wrong:** Clerk user IDs are strings (e.g., `user_2abc...`), not UUIDs. If `saved_jobs.user_id` is typed as `uuid` in Postgres, inserts fail with a type error.

**Why it happens:** Developers default all ID columns to `uuid` without checking the type of Clerk-provided IDs.

**How to avoid:** Type `user_id` columns as `text` (not `uuid`) in Drizzle schema when storing Clerk user IDs. Only use `uuid` for internally-generated IDs.

### Pitfall 5: Not Seeding Job Listings Before Testing

**What goes wrong:** The paginated job list page renders an empty state immediately. There is no "browse" experience to validate.

**Why it happens:** Developers build the data model and UI but skip seeding.

**How to avoid:** Create a `seed.ts` script (using Drizzle) with 10-20 realistic job listings covering a range of `posted_at` dates (including some older than 30 days to validate staleness display). Run it once after initial migration.

---

## Code Examples

Verified patterns from the stack's official sources:

### Clerk Middleware Setup (from Clerk official docs)

```typescript
// middleware.ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server'

const isPublicRoute = createRouteMatcher(['/sign-in(.*)', '/sign-up(.*)', '/jobs(.*)'])

export default clerkMiddleware(async (auth, req) => {
  if (!isPublicRoute(req)) await auth.protect()
})

export const config = {
  matcher: ['/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jinja2|txt|xml|ico|webp|png|jpg|gif|svg|ttf|woff2?|eot|otf|wasm)).*)', '/(api|trpc)(.*)'],
}
```

### Drizzle DB Singleton (Vercel-safe)

```typescript
// lib/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'
import * as schema from './schema'

const client = postgres(process.env.DATABASE_URL!, { max: 1 })  // max:1 for serverless
export const db = drizzle(client, { schema })
```

### Paginated Job Query

```typescript
// lib/queries/jobs.ts
import { db } from '@/lib/db'
import { jobs } from '@/lib/db/schema'
import { desc, sql, count } from 'drizzle-orm'

export async function getJobs(page = 1, limit = 20) {
  const offset = (page - 1) * limit
  const [rows, [{ total }]] = await Promise.all([
    db.select({
      id: jobs.id,
      title: jobs.title,
      company: jobs.company,
      location: jobs.location,
      roleType: jobs.roleType,
      postedAt: jobs.postedAt,
      commentCount: jobs.commentCount,
      isStale: sql<boolean>`${jobs.postedAt} < NOW() - INTERVAL '30 days'`,
    })
    .from(jobs)
    .orderBy(desc(jobs.lastActivityAt))
    .limit(limit)
    .offset(offset),
    db.select({ total: count() }).from(jobs),
  ])
  return { jobs: rows, total, page, totalPages: Math.ceil(total / limit) }
}
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| NextAuth.js v4 | Clerk (managed) or Better Auth (self-hosted) | 2025-2026 | NextAuth.js is in maintenance; Auth.js v5 never stabilized; ecosystem has consolidated |
| Prisma (Rust engine) | Drizzle ORM 0.38.x | 2023-2024 | Drizzle has no cold-start penalty on Vercel serverless; smaller bundle |
| Tailwind CSS 3.x (tailwind.config.js) | Tailwind CSS 4.x (CSS-native @theme) | 2025 | Config file eliminated; shadcn/ui confirmed v4 support |
| `next-auth` session provider wrapping entire app | Clerk `<ClerkProvider>` in root layout | 2025 | Clerk provider is lighter; session is available server-side via `auth()` without client wrapper |

**Deprecated/outdated:**
- `next-auth` v4: Maintenance mode only; do not use for new projects
- Auth.js v5 (`next-auth@beta`): Never reached stable; community moved on
- `@auth/drizzle-adapter`: Only relevant with Auth.js, which is deprecated — not needed with Clerk

---

## Open Questions

1. **Listing supply strategy (unresolved blocker from STATE.md)**
   - What we know: Phase 1 needs job listings to seed the browse experience; the data model is ready
   - What's unclear: Source of listings — employer-direct, third-party aggregator API, or manual curation
   - Recommendation: Manually curate 20-50 listings via a seed script for Phase 1; defer API aggregator decision to Phase 3. This unblocks implementation without requiring the business/legal decision to be made first.

2. **LinkedIn OAuth setup time**
   - What we know: Clerk supports LinkedIn OAuth via dashboard toggle
   - What's unclear: LinkedIn's developer app review process can take days; this should not block Phase 1 but must be started early
   - Recommendation: Create LinkedIn developer app on day 1 of implementation and submit for review; use Google OAuth as the fallback during the wait.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | None detected yet — Wave 0 must establish |
| Config file | None — see Wave 0 |
| Quick run command | `npx vitest run --reporter=verbose` (after Wave 0 setup) |
| Full suite command | `npx vitest run` |

Rationale for Vitest: Native ESM support, compatible with Next.js 15 and TypeScript 5, fast HMR-aware test runner. Jest requires additional transform config for ESM and RSC patterns.

### Phase Requirements to Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| AUTH-01 | Email/password sign-up creates account | Manual (Clerk handles; no custom logic to unit test) | — | — |
| AUTH-02 | Google OAuth sign-in redirects and resolves session | Manual (OAuth flow cannot be unit tested without mocking Clerk) | — | — |
| AUTH-03 | LinkedIn OAuth sign-in redirects and resolves session | Manual (same as AUTH-02) | — | — |
| AUTH-04 | Session cookie persists across page refresh | Manual (browser test; Clerk's responsibility) | — | — |
| AUTH-05 | Sign-out clears session from all pages | Manual (Clerk's `signOut()` function) | — | — |
| JOBS-01 | `getJobs()` returns paginated rows with correct offset | Unit | `npx vitest run tests/queries/jobs.test.ts` | Wave 0 |
| JOBS-02 | `getJobById()` returns job or null for unknown id | Unit | `npx vitest run tests/queries/jobs.test.ts` | Wave 0 |
| JOBS-03 | `saveJob()` inserts row; duplicate insert is idempotent | Unit | `npx vitest run tests/actions/jobs.test.ts` | Wave 0 |
| JOBS-04 | `unsaveJob()` removes row; re-delete is a no-op | Unit | `npx vitest run tests/actions/jobs.test.ts` | Wave 0 |
| JOBS-05 | `isStale` is true when `posted_at` is 31+ days ago | Unit | `npx vitest run tests/queries/jobs.test.ts` | Wave 0 |

Note: AUTH requirements are tested manually because they depend on Clerk's hosted UI and OAuth provider infrastructure. The custom logic surface area for auth is intentionally minimal (middleware route matching only).

### Sampling Rate

- **Per task commit:** `npx vitest run tests/queries/jobs.test.ts tests/actions/jobs.test.ts`
- **Per wave merge:** `npx vitest run`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `tests/queries/jobs.test.ts` — covers JOBS-01, JOBS-02, JOBS-05
- [ ] `tests/actions/jobs.test.ts` — covers JOBS-03, JOBS-04
- [ ] `vitest.config.ts` — Vitest configuration with path aliases matching `tsconfig.json`
- [ ] `tests/helpers/db.ts` — test database setup/teardown using a test schema or in-memory SQLite via Drizzle
- [ ] Framework install: `npm install -D vitest @vitest/ui`

---

## Sources

### Primary (HIGH confidence)

- [Clerk Next.js Quickstart](https://clerk.com/docs/quickstarts/nextjs) — Middleware setup, ClerkProvider, auth() usage
- [Clerk pricing page](https://clerk.com/pricing) — 10,000 MAU free tier confirmed
- [Clerk LinkedIn OAuth docs](https://clerk.com/docs/authentication/social-connections/linkedin) — LinkedIn OAuth configuration
- [Drizzle ORM PostgreSQL guide](https://orm.drizzle.team/docs/get-started-postgresql) — Schema definition, query patterns
- [Drizzle ORM v1 roadmap](https://orm.drizzle.team/roadmap) — v1.0 still beta; stable 0.38.x recommended
- [shadcn/ui Tailwind v4 docs](https://ui.shadcn.com/docs/tailwind-v4) — Tailwind v4 compatibility confirmed
- [Next.js 15 Server Actions docs](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations) — `'use server'` directive, revalidatePath
- [Auth.js joins Better Auth announcement](https://better-auth.com/blog/authjs-joins-better-auth) — NextAuth.js deprecation confirmed

### Secondary (MEDIUM confidence)

- [Stream.io: Scalable Activity Feed Architecture](https://getstream.io/blog/scalable-activity-feed-architecture/) — Pull-based feed rationale
- [Modern Full-Stack Next.js 15 Architecture (SoftwareMill)](https://softwaremill.com/modern-full-stack-application-architecture-using-next-js-15/) — Server/Client component boundaries
- [Designing a Database for an Online Job Portal (Redgate)](https://www.red-gate.com/blog/designing-a-database-for-an-online-job-portal/) — Job listing schema entities

### Tertiary (LOW confidence — validate during implementation)

- Drizzle serverless connection pooling (`max: 1`) — sourced from community documentation; verify against current Drizzle docs at implementation time

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — Clerk, Drizzle, Next.js 15, Tailwind 4 all confirmed via official sources with current documentation
- Architecture: HIGH — patterns are canonical Next.js App Router patterns with official documentation support
- Pitfalls: HIGH — Clerk env variable and Drizzle singleton pitfalls are well-documented in official troubleshooting guides; UUID type mismatch is a known Clerk+Drizzle integration issue

**Research date:** 2026-03-04
**Valid until:** 2026-06-04 (stable stack; Clerk and Drizzle APIs are unlikely to have breaking changes within 90 days)
