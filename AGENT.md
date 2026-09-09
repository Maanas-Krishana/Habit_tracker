# AGENT.md

Guidance for AI coding agents (e.g. Claude Code) working in this repository.

## Project Summary
A web + mobile habit tracker. Core features: streak counter, GitHub-style contribution calendar (monthly/annual views), activity badges, and a secure public API that lets third parties embed a user's activity graph/badges elsewhere. See `ACTION_PLAN.md` for the full versioned roadmap.

## Roadmap Snapshot
- **v1 (current target):** habits CRUD, activity logging, streak calculation, contribution calendar (web + app)
- **v2:** authentication, dark mode, activity badges (weekly/monthly), secure scoped public API
- **v3:** Android home-screen widget (pending activity to-do), web admin dashboard (usage analytics)

Agents should default to working on the **earliest incomplete version** unless told otherwise — don't build v3 features before v1/v2 are solid.

## Repository Structure (proposed — adjust as it firms up)
```
/backend        # API server, business logic, DB models
/web            # Web frontend
/mobile         # App frontend (iOS/Android shell)
/widget         # Android widget module (v3)
/admin          # Admin dashboard (v3, may live inside /web as a route)
/docs           # API docs (OpenAPI spec, etc.)
ACTION_PLAN.md  # Roadmap and phase breakdown
AGENT.md        # This file
```

## Core Domain Rules (must hold across the whole codebase)
1. **Streaks are derived from `activity_logs`, never hand-edited.** Any streak-fixing bug should be fixed in the calculation function, not by patching stored streak values directly.
2. **Timezone handling is centralized.** All "what day is it" logic goes through one shared utility — never call `new Date()` / equivalent directly in streak, calendar, or badge code.
3. **The public API (v2+) is read-only and scoped.** A public API key must never be able to access anything beyond the activity graph and badges for the user who issued it. Treat any change to this surface as security-sensitive.
4. **Badge rules live in one place.** Don't scatter badge-eligibility logic across frontend and backend — compute badges server-side and treat the client as a display layer only.

## Conventions
- **API versioning:** all public-facing endpoints are prefixed (`/v1/`, `/v2/`); do not introduce breaking changes to a released version — add a new version instead.
- **Testing:** streak calculation and badge-eligibility logic require unit tests covering edge cases (missed day, backfilled entry, timezone rollover, leap years/DST) before merging.
- **Commits/PRs:** reference the roadmap version a change belongs to (e.g. `[v2] add badge eligibility check`).

## What Agents Should Do Before Big Changes
- Check `ACTION_PLAN.md` for which version a requested feature belongs to.
- If a request would jump ahead of the current version (e.g. building the admin dashboard before auth exists), flag this rather than silently building it.
- For anything touching the public API's auth/scoping, treat it as security-sensitive: prefer smaller, reviewable diffs and explicit tests over broad changes.

## Open Decisions (not yet settled — ask before assuming)
- Backend framework / language
- Database choice
- Mobile framework (React Native, Flutter, native)
- Badge rules: hardcoded set vs. configurable rules engine
- Auth provider: third-party (Auth0/Clerk/etc.) vs. custom JWT implementation
