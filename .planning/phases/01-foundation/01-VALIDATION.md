---
phase: 1
slug: foundation
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-04
---

# Phase 1 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | vitest + @testing-library/react |
| **Config file** | vitest.config.ts — Wave 0 installs |
| **Quick run command** | `npx vitest run --reporter=verbose` |
| **Full suite command** | `npx vitest run` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run `npx vitest run --reporter=verbose`
- **After every plan wave:** Run `npx vitest run`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| auth-setup | 01 | 1 | AUTH-01–05 | integration | `npx vitest run auth` | ❌ W0 | ⬜ pending |
| db-schema | 01 | 1 | JOBS-01–05 | unit | `npx vitest run schema` | ❌ W0 | ⬜ pending |
| jobs-list | 01 | 2 | JOBS-01, JOBS-02 | unit | `npx vitest run jobs` | ❌ W0 | ⬜ pending |
| saved-jobs | 01 | 2 | JOBS-03, JOBS-04 | unit | `npx vitest run saved` | ❌ W0 | ⬜ pending |
| staleness | 01 | 2 | JOBS-05 | unit | `npx vitest run stale` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `tests/auth.test.ts` — stubs for AUTH-01–05
- [ ] `tests/schema.test.ts` — Drizzle schema shape validation
- [ ] `tests/jobs.test.ts` — job listing query stubs
- [ ] `tests/saved.test.ts` — saved jobs CRUD stubs
- [ ] `tests/stale.test.ts` — staleness predicate unit test
- [ ] `vitest.config.ts` — test framework config
- [ ] `npm install -D vitest @testing-library/react @vitejs/plugin-react`

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Google OAuth sign-in flow | AUTH-02 | Requires live OAuth provider | Click "Sign in with Google", complete flow, verify session |
| LinkedIn OAuth sign-in flow | AUTH-03 | Requires live OAuth provider + LinkedIn app review | Click "Sign in with LinkedIn", complete flow, verify session |
| Session persists on refresh | AUTH-04 | Browser state | Sign in, refresh page, verify still logged in |
| Sign out from any page | AUTH-05 | UI interaction | Sign out from 3+ different pages, verify redirect |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
