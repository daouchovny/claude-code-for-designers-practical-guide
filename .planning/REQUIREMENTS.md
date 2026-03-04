# Requirements: CareerSignal

**Defined:** 2026-03-04
**Core Value:** Mid-career switchers land roles by finding opportunities the community has already talked about, reviewed, and validated — not by sending applications into a void.

## v1 Requirements

### Authentication

- [ ] **AUTH-01**: User can create an account with email and password
- [ ] **AUTH-02**: User can sign in with Google OAuth
- [ ] **AUTH-03**: User can sign in with LinkedIn OAuth
- [ ] **AUTH-04**: User session persists across browser refresh
- [ ] **AUTH-05**: User can sign out from any page

### Jobs

- [ ] **JOBS-01**: User can browse a paginated list of job listings
- [ ] **JOBS-02**: User can view a job listing detail page (title, company, description, location, role type)
- [ ] **JOBS-03**: User can save a job listing to their saved jobs
- [ ] **JOBS-04**: User can view and manage their saved jobs list
- [ ] **JOBS-05**: Job listings display posted date and are marked stale after 30 days

### Community

- [ ] **COMM-01**: User can post a comment on a job listing
- [ ] **COMM-02**: User can reply to an existing comment on a job listing (max 1 reply depth)
- [ ] **COMM-03**: User can view all community discussion on a job listing
- [ ] **COMM-04**: Comments display author's career context (current role, target role if set)

### Discovery

- [ ] **DISC-01**: User can search job listings by keyword (title, company, skill)
- [ ] **DISC-02**: User can filter search results by role type (full-time, contract, part-time)
- [ ] **DISC-03**: User can filter search results by location / work style (remote, hybrid, in-office)
- [ ] **DISC-04**: Search results display community discussion count as a signal of vetted interest

## v2 Requirements

### Networking

- **NETW-01**: User can follow other users
- **NETW-02**: User can view other users' public profiles
- **NETW-03**: User can see activity from people they follow in a feed

### Personalization

- **PERS-01**: User can set target role and career switch goals on their profile
- **PERS-02**: User receives a personalized job feed ranked by relevance to their target role (not past experience)
- **PERS-03**: User can follow companies and get notified of new listings

### Community Depth

- **CDEP-01**: User can upvote comments to surface the most useful intel
- **CDEP-02**: Structured intel prompts guide community contributions (e.g., "What's the interview process like?")
- **CDEP-03**: Users can see salary data contributed anonymously by community members

### Employer Side

- **EMPL-01**: Employers can create an account and post job listings
- **EMPL-02**: Employers can view aggregate community engagement on their listings (read-only)
- **EMPL-03**: Employer accounts have no ability to edit or suppress community content

### Notifications

- **NOTF-01**: User receives in-app notifications for replies to their comments
- **NOTF-02**: User receives email digest of new discussion on saved jobs (weekly)
- **NOTF-03**: User can configure notification preferences

## Out of Scope

| Feature | Reason |
|---------|--------|
| Direct messaging between users | Creates LinkedIn spam dynamic; out of scope to protect community quality |
| Quick apply / one-click apply | Trains spray-and-pray behavior; counter to the platform's intentional discovery model |
| AI resume builder | Not the core value; dilutes focus on community discovery |
| Mobile native app (iOS/Android) | Web-first to validate core value; mobile follows after PMF |
| Anonymous posting | Increases moderation burden and reduces trust signals; professional identity is part of the value |
| Video interviews / integrations | Too early; not relevant to community discovery model |
| Courses / skill-building content | Separate product surface; outside the job discovery scope |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 1 | Pending |
| AUTH-02 | Phase 1 | Pending |
| AUTH-03 | Phase 1 | Pending |
| AUTH-04 | Phase 1 | Pending |
| AUTH-05 | Phase 1 | Pending |
| JOBS-01 | Phase 1 | Pending |
| JOBS-02 | Phase 1 | Pending |
| JOBS-03 | Phase 1 | Pending |
| JOBS-04 | Phase 1 | Pending |
| JOBS-05 | Phase 1 | Pending |
| COMM-01 | TBD | Pending |
| COMM-02 | TBD | Pending |
| COMM-03 | TBD | Pending |
| COMM-04 | TBD | Pending |
| DISC-01 | TBD | Pending |
| DISC-02 | TBD | Pending |
| DISC-03 | TBD | Pending |
| DISC-04 | TBD | Pending |

**Coverage:**
- v1 requirements: 18 total
- Mapped to phases: 10 (traceability to be completed by roadmapper)
- Unmapped: 8 ⚠️ (to be resolved during roadmap creation)

---
*Requirements defined: 2026-03-04*
*Last updated: 2026-03-04 after initial definition*
