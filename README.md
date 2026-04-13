# Claude Code for Designers: Practical Guide

A hands-on guide for designers learning to build with Claude Code. This repository follows the development of **CareerSignal** — a community-driven job discovery platform for mid-career switchers — built entirely through Claude Code sessions.

## What This Is

This repo documents the process of going from requirements and research to a working web application using Claude Code as the primary development tool. It's structured so designers can follow along, replicate the approach, and adapt it to their own projects.

The example app, **CareerSignal**, lets mid-career professionals discover job listings enriched with community discussion and peer intel — making every role feel vetted before you apply.

## Technologies

| Layer | Technology |
|-------|------------|
| Framework | [Next.js 15](https://nextjs.org) (App Router, Server Actions) |
| Language | TypeScript 5.x |
| UI | React 19, [shadcn/ui](https://ui.shadcn.com), [Tailwind CSS 4](https://tailwindcss.com) |
| Database | PostgreSQL 16 |
| ORM | [Drizzle ORM](https://orm.drizzle.team) 0.38.x |
| Auth | [Clerk](https://clerk.com) |
| Client state | [TanStack Query](https://tanstack.com/query) 5.x |
| Forms | React Hook Form + Zod |
| Email | [Resend](https://resend.com) + React Email |
| Caching / rate limiting | [Upstash Redis](https://upstash.com) |
| Error tracking | [Sentry](https://sentry.io) |
| Hosting | [Vercel](https://vercel.com) |

## Running Locally

### Prerequisites

- Node.js 18.18.0 or later
- A PostgreSQL database (local or hosted via [Neon](https://neon.tech) / [Supabase](https://supabase.com))
- A [Clerk](https://clerk.com) account (free tier)
- A [Upstash](https://upstash.com) Redis database (free tier)

### Setup

1. Clone the repo

```bash
git clone https://github.com/daouchovny/claude-code-for-designers-practical-guide.git
cd claude-code-for-designers-practical-guide/CareerSignal
```

2. Install dependencies

```bash
npm install
```

3. Copy the environment template and fill in your credentials

```bash
cp .env.example .env.local
```

Required environment variables:

```
# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=

# Database
DATABASE_URL=

# Upstash Redis
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=
```

4. Push the database schema

```bash
npx drizzle-kit push
```

5. Start the dev server

```bash
npm run dev
```

The app will be available at [http://localhost:3000](http://localhost:3000).

## Project Structure

```
.planning/          # Requirements, roadmap, and research (Claude Code planning docs)
CareerSignal/       # The Next.js application
```

## Build Roadmap

| Phase | Goal | Status |
|-------|------|--------|
| 1. Foundation | Auth + job listing inventory | Not started |
| 2. Community | Threaded discussion on job listings | Not started |
| 3. Discovery | Search, filters, community signal | Not started |
