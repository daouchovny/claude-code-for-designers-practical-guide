# Roadmap: CareerSignal

## Overview

CareerSignal ships in three phases, each delivering a coherent and verifiable capability. Phase 1 establishes identity and the job listing inventory — the two prerequisites everything else depends on. Phase 2 delivers the core differentiator: community discussion layered onto listings. Phase 3 makes jobs findable and surfaces community signal as a quality indicator. The sequence is architecturally constrained: discussion has no anchor without listings, and search signal is meaningless without discussion to count.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Foundation** - Users can create accounts and browse a live job listing inventory
- [ ] **Phase 2: Community** - Users can read and contribute community discussion on job listings
- [ ] **Phase 3: Discovery** - Users can find relevant jobs through search, filters, and community signal

## Phase Details

### Phase 1: Foundation
**Goal**: Users can securely access their accounts and browse a vetted job listing inventory
**Depends on**: Nothing (first phase)
**Requirements**: AUTH-01, AUTH-02, AUTH-03, AUTH-04, AUTH-05, JOBS-01, JOBS-02, JOBS-03, JOBS-04, JOBS-05
**Success Criteria** (what must be TRUE):
  1. User can create an account with email/password and sign in with Google or LinkedIn OAuth
  2. User session persists across browser refresh and user can sign out from any page
  3. User can browse a paginated list of job listings and view full listing detail pages
  4. User can save a job listing and manage their saved jobs list
  5. Job listings display posted date and are visibly marked stale after 30 days
**Plans**: TBD

### Phase 2: Community
**Goal**: Users can read and contribute threaded community discussion on job listings, with author career context visible
**Depends on**: Phase 1
**Requirements**: COMM-01, COMM-02, COMM-03, COMM-04
**Success Criteria** (what must be TRUE):
  1. User can post a top-level comment on any job listing
  2. User can reply to an existing comment (one level deep — no replies to replies)
  3. User can read the full community discussion thread on any listing
  4. Comments display the author's career context (current role and target role if set)
**Plans**: TBD

### Phase 3: Discovery
**Goal**: Users can find jobs through keyword search and filters, with community engagement visible as a signal of vetted interest
**Depends on**: Phase 2
**Requirements**: DISC-01, DISC-02, DISC-03, DISC-04
**Success Criteria** (what must be TRUE):
  1. User can search job listings by keyword (title, company, or skill)
  2. User can filter search results by role type (full-time, contract, part-time)
  3. User can filter search results by location or work style (remote, hybrid, in-office)
  4. Search results display the community discussion count alongside each listing
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 0/TBD | Not started | - |
| 2. Community | 0/TBD | Not started | - |
| 3. Discovery | 0/TBD | Not started | - |
