# Feature Research

**Domain:** Professional networking / community-driven job discovery (mid-career switchers)
**Researched:** 2026-03-04
**Confidence:** MEDIUM — primary sources are competitor analysis + industry research; no direct user research data available

## Feature Landscape

### Table Stakes (Users Expect These)

Features users assume exist. Missing these = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| User profile with career history | Every professional platform has this; without it, users have no identity or credibility | MEDIUM | Must show career trajectory and target roles, not just current job — this is the mid-career pivot context |
| Job listings (browsable + searchable) | Core product purpose; users come expecting to find jobs | HIGH | Needs either employer-direct postings or backfilled listings from external sources to avoid cold-start emptiness |
| Company pages with basic info | Users validate companies before discussing or applying | MEDIUM | Name, size, industry, location, description minimum. Links to Glassdoor/LinkedIn are not sufficient — need native context |
| Search and filter for jobs | Users expect to filter by location, role, industry, salary range | MEDIUM | Filters must map to mid-career switcher mental models (e.g., "roles open to career changers") not just standard LinkedIn filters |
| Discussion threads on job listings | This is CareerSignal's core mechanic, but users will expect it once they see the first listing | HIGH | Threading model (reply chains vs. flat comments) needs an early design decision |
| Discussion threads on companies | Glassdoor/Blind have trained users to expect candid company commentary | HIGH | Must be distinct from formal reviews — conversational tone, not survey-style |
| User authentication and account management | Non-negotiable baseline | LOW | Email + social login (Google, LinkedIn OAuth) expected |
| Notifications for new activity | Follow a job, a company, a topic — users expect to be alerted | MEDIUM | Email + in-app minimum; push notifications for mobile later |
| Basic onboarding flow | Mid-career switchers have complex backgrounds; onboarding must capture what they're moving FROM and what they're moving TO | MEDIUM | Profile completeness gate needed to drive community quality |
| Mobile-responsive web | Users browse jobs on phones; non-responsive = immediate bounce | LOW | Web-first is right call, but must be fully responsive from day one |

### Differentiators (Competitive Advantage)

Features that set the product apart. Not required, but valued.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Community-vetted job signal (upvotes, engagement indicators) | Mid-career switchers don't want to read 300 listings; community activity signals which roles are real, interesting, or worth pursuing | HIGH | This is the core differentiator — a listing with 12 comments and 47 upvotes reads differently than a blank listing. Requires critical mass of users to work. |
| Salary transparency on listings | Levels.fyi proved crowdsourced salary data is one of the most powerful retention hooks in this space; users return repeatedly to check data | HIGH | Crowdsourced submission model (anonymous, verified-optional) works better than requiring employer disclosure |
| "Is this role open to career changers?" signal | The #1 anxiety of mid-career switchers is whether they'll be screened out; surfacing which roles have hired career changers is a novel filter no competitor offers | MEDIUM | Can start as a community-tagged signal — users report their transition path when sharing intel |
| Career switcher success stories attached to listings | "Someone went from teacher to UX designer using this company's apprenticeship" is more useful than any job description copy | MEDIUM | Content moderation challenge — needs curation, not just open submission |
| Peer validation threads: "Is this role worth applying to?" | Turns a passive job board into an active advice community; mirrors how mid-career switchers actually use Reddit (r/cscareerquestions) but scoped to specific listings | HIGH | The community mechanic that makes browsing feel like getting advice from a friend who already did the research |
| Company culture intel from community members (not anonymous reviews) | Glassdoor reviews feel stale and adversarial; Fishbowl/Blind feel too anonymous and chaotic. Verified-employee commentary attached to a profile builds more credibility | HIGH | Verification via work email (Blind model) reduces noise significantly. Requires trust architecture. |
| Personalized job recommendations based on target role + background | LinkedIn's recommendations are too broad; mid-career switchers need suggestions that account for transferable skills, not just keyword matches | HIGH | ML/AI component — defer to v1.x unless using an existing recommendation API |
| Follow other professionals on their career journey | "Following" a senior UX designer who switched from architecture gives mid-career switchers a roadmap and role model | MEDIUM | Asymmetric follow model (Twitter-style) rather than mutual connections (LinkedIn-style) lowers the anxiety of reaching out |
| Employer/recruiter transparency scores | Which companies actually respond to applicants? Which ghost? Community-reported response rates build a layer of employer accountability not found on job boards | MEDIUM | Gameable if not designed carefully — needs minimum submission thresholds before display |
| Saved job collections + personal notes | Users research over weeks or months; ability to tag, annotate, and track listings is a retention feature | LOW | Simple MVP version is just bookmarks; full version is a personal CRM for job hunting |

### Anti-Features (Commonly Requested, Often Problematic)

Features that seem good but create problems.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Direct messaging between strangers (cold DMs) | Users want to reach hiring managers or ask advice from strangers | Creates immediate LinkedIn-style recruiter spam problem; destroys community feel; mid-career switchers will receive unsolicited outreach that erodes trust | Public discussion threads where you can @mention and reply — keeps conversations visible to the community, reduces spam incentive |
| Fully anonymous posting | Users want to share sensitive intel without professional risk | Anonymous platforms (Blind) consistently attract harassment, bad-faith posts, and toxic threads that require heavy moderation investment; anonymity and quality are in tension | Pseudonymous with verified credentials — users choose a display name but their industry and seniority level are verified, preserving some safety without full anonymity |
| AI resume builder / resume optimization | Users ask for it because the job search is painful | Scope creep away from community-first differentiation; every job board has this now, making it a commodity feature that dilutes the brand | Link out to specialized tools (Jobscan, etc.) or add a community feature: "What resume approach worked for your industry switch?" |
| One-click / quick-apply to all listings | Reduces friction, users will ask for it | Mass-apply behavior produces 0% response rates (Jobscan data: 60% of applications go unseen); it trains users to spam-apply rather than making informed decisions — antithetical to community-vetted job discovery | Community-informed apply: prompt users to read the discussion thread before applying; surface "what helped people get callbacks here" |
| Employer ratings (star ratings, 1-5 scale) | Feels like Glassdoor, seems obvious | Aggregate star ratings invite gaming, legal challenges, and gaming by employer reputation management; also reduces nuanced community insight to a meaningless number | Qualitative community threads by topic (culture, growth, management) are richer and harder to game |
| Real-time chat / live messaging rooms | Creates engagement, feels modern | High moderation burden; requires constant seeding to prevent ghost-town feel; threads work better for async community knowledge-building | Threaded discussions on listings and companies are the right default; live chat is a distraction at MVP scale |
| Course recommendations / skill-building curriculum | Mid-career switchers need to learn new skills; feels complementary | Education is a fully separate domain with established competitors (Coursera, LinkedIn Learning); building it is a multi-year investment that pulls focus | Surface what skills community members say they needed for their transitions in discussion threads; link out to resources users recommend |

## Feature Dependencies

```
[User Profile]
    └──required by──> [Community Discussion]
    └──required by──> [Follow / Connections]
    └──required by──> [Personalized Recommendations]
    └──required by──> [Employer Intel Contributions]

[Job Listings]
    └──required by──> [Discussion Threads on Listings]
    └──required by──> [Community-Vetted Signal / Upvotes]
    └──required by──> [Saved Job Collections]
    └──required by──> [Salary Transparency on Listings]

[Company Pages]
    └──required by──> [Company Culture Threads]
    └──required by──> [Employer Transparency Scores]

[Discussion Threads on Listings]
    └──enhanced by──> [Community-Vetted Signal] (upvotes surface best threads)
    └──enhanced by──> [User Profile] (credibility of who is commenting)
    └──enhanced by──> [Notifications] (drives engagement loops)

[User Auth]
    └──required by──> [User Profile]
    └──required by──> [All community features]

[Critical Mass of Users]
    └──required by──> [Community-Vetted Signal] (needs enough activity to mean something)
    └──required by──> [Salary Transparency] (needs enough submissions to be credible)
    └──required by──> [Employer Transparency Scores] (needs enough reports)

[Anonymous / Verified-Employee Discussion]
    └──conflicts with──> [Direct Messaging between strangers]
    (one requires identity exposure, the other requires safety from it)
```

### Dependency Notes

- **User Profile requires Auth:** Trivially true but important — onboarding quality gates (career history, target roles) must be solved before community features launch, or discussions will feel empty.
- **Community-Vetted Signal requires Critical Mass:** This is the platform's biggest dependency risk. The differentiating feature doesn't work until there are enough users. Cold-start strategy must address this explicitly — likely through backfilling job listings and curating early community members.
- **Job Listings requires supply strategy:** Either employer-direct postings or backfilled listings from external sources. Without listings, the discussion mechanic has nothing to attach to. This is the chicken-and-egg problem specific to CareerSignal.
- **Salary Transparency requires minimum data threshold:** Following Levels.fyi's model, suppress data until there are 5+ submissions for a company/role combination. Displaying n=1 data is misleading.

## MVP Definition

### Launch With (v1)

Minimum viable product — what's needed to validate the concept.

- [ ] User profile with career trajectory and target roles — needed before any community feature has credibility
- [ ] Job listings (backfilled from external sources) — solves cold-start on the supply side; gives users something to discuss
- [ ] Company pages (basic info, auto-generated from listings) — anchors company-level discussion
- [ ] Discussion threads on job listings — the core mechanic; this is what CareerSignal does that no one else does
- [ ] Discussion threads on company pages — complements listing threads; users naturally move between the two
- [ ] Search and filter (role, location, industry) — users cannot engage with jobs they cannot find
- [ ] User auth (email + Google OAuth) — baseline
- [ ] Notifications for thread replies and new listings in followed companies/roles — closes engagement loops
- [ ] Mobile-responsive design — non-negotiable from day one

### Add After Validation (v1.x)

Features to add once core is working.

- [ ] Community-vetted signal (upvotes on listings, discussion activity scores) — add when there's enough volume that signal is meaningful (rough threshold: 50+ active weekly users)
- [ ] Saved job collections + personal notes — add when users show repeat-visit behavior; signals job tracking is happening
- [ ] Salary transparency (crowdsourced submissions) — add once there's an active community willing to contribute; premature launch produces misleading sparse data
- [ ] Follow other professionals — add once profiles feel complete and users have reason to track each other's trajectories
- [ ] Employer/recruiter response rate tracking — add after employer side is established; needs minimum data volume to be credible

### Future Consideration (v2+)

Features to defer until product-market fit is established.

- [ ] Personalized job recommendations (ML-based) — defer; requires data volume and engineering investment not justified pre-PMF
- [ ] "Is this role open to career changers?" signal — defer implementation as a formal feature; can be captured informally in discussion threads first, then formalized
- [ ] Employer/recruiter portal (posting + community engagement) — defer; brings monetization complexity and changes community dynamics before trust is built
- [ ] Mobile native app — defer; web-first validates faster

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| User profile (career trajectory + target roles) | HIGH | MEDIUM | P1 |
| Job listings (backfilled) | HIGH | MEDIUM | P1 |
| Discussion threads on listings | HIGH | HIGH | P1 |
| Discussion threads on companies | HIGH | MEDIUM | P1 |
| User auth | HIGH | LOW | P1 |
| Search and filter | HIGH | MEDIUM | P1 |
| Mobile-responsive design | HIGH | LOW | P1 |
| Notifications | MEDIUM | MEDIUM | P1 |
| Company pages (basic) | MEDIUM | LOW | P1 |
| Community-vetted signal (upvotes) | HIGH | MEDIUM | P2 |
| Saved job collections | MEDIUM | LOW | P2 |
| Salary transparency (crowdsourced) | HIGH | HIGH | P2 |
| Follow other professionals | MEDIUM | MEDIUM | P2 |
| Employer response rate tracking | MEDIUM | HIGH | P2 |
| Career switcher success stories | HIGH | MEDIUM | P2 |
| Personalized recommendations (ML) | HIGH | HIGH | P3 |
| Employer/recruiter portal | MEDIUM | HIGH | P3 |
| Native mobile app | MEDIUM | HIGH | P3 |

**Priority key:**
- P1: Must have for launch
- P2: Should have, add when possible
- P3: Nice to have, future consideration

## Competitor Feature Analysis

| Feature | LinkedIn | Glassdoor/Fishbowl | Blind/Levels.fyi | CareerSignal Approach |
|---------|--------------|--------------|--------------|--------------|
| Job listings | Extensive, employer-direct | Basic, linked to reviews | Limited (Levels has some) | Backfilled + employer-direct over time |
| Community discussion | Noisy feed, not attached to listings | Reviews attached to companies (not listings), Fishbowl bowls | Blind: anonymous forums; Levels: salary threads | Threaded discussions attached directly to listings AND companies |
| Salary transparency | Hidden, negotiation-based | Basic salary ranges | Core feature (Levels.fyi), crowdsourced and verified | v1.x feature, crowdsourced model following Levels.fyi |
| Anonymity model | Real name, low candor | Glassdoor: semi-anon reviews; Fishbowl: pseudonymous | Blind: work-email verified anon | Verified credentials, pseudonymous display — credibility without full exposure |
| Career switcher focus | None (everyone is the audience) | None | None | Core — targeting mid-career switchers explicitly |
| Community signal on listings | Algorithmic, opaque | None | Upvotes in community threads | Upvotes + engagement activity score surfaced on listing cards |
| Direct messaging | Yes, major spam problem | No | No | Deliberately excluded from v1 |
| Profile model | Reverse-chronological career history | Minimal | Minimal | Career trajectory emphasis — where you've been AND where you're going |

## Sources

- [Levels.fyi — Salary transparency model analysis](https://www.levels.fyi/)
- [How Levels.fyi Brought Salary Transparency to the Tech Industry](https://www.tryexponent.com/blog/levels-interview-tech-salary-transparency)
- [Glassdoor acquires Fishbowl — community features acquisition rationale](https://techcrunch.com/2021/09/14/glassdoor-fishbowl-linkedin/)
- [Glassdoor introduces anonymous community features](https://techcrunch.com/2023/07/18/glassdoor-is-introducing-blind-like-anonymous-community-features-to-fuel-user-growth/)
- [LinkedIn Alternatives for Job Seekers in 2025](https://www.tryapt.ai/blog/linkedin-alternatives-for-job-seekers-in-2025)
- [Career Switchers: The Most Valuable Talent Pool in 2026](https://idealtraits.com/blog/career-switchers-the-most-valuable-talent-pool-in-2026/)
- [Job Search in 2025: Why 6 in 10 Applications Go Unseen](https://www.myperfectresume.com/career-center/careers/basics/job-search-behavior) — supports anti-feature case against quick-apply
- [Job Boards: A Path to Higher Engagement and New Memberships](https://www.higherlogic.com/blog/job-boards-and-career-communities-a-path-to-higher-engagement-and-new-memberships/) — community + job board integration patterns
- [Cracking the chicken and egg problem for job boards](https://www.smartjobboard.com/blog/chicken-egg-problem-for-job-boards/) — cold-start strategy
- [The State of the Job Search in 2025 — Jobscan](https://www.jobscan.co/state-of-the-job-search)
- [Best Professional Networking Platforms 2026 — Product Hunt](https://www.producthunt.com/categories/professional-networking)
- [Blind Platform — anonymous professional community model](https://www.teamblind.com/)

---
*Feature research for: CareerSignal — professional networking / community-driven job discovery*
*Researched: 2026-03-04*
