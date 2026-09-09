# Habit Tracker — Action Plan

## Overview
A web + mobile habit tracker with a streak counter and a GitHub-style contribution calendar (monthly + annual views). Roadmap spans three versions: core tracking (v1), auth/badges/public API (v2), and Android widget/admin analytics (v3).

---

## Version 1 — Core Product
**Goal:** Ship a working single-user habit tracker.

1. **Data model**
   - `users` (even if single-user for now, design for multi-user from day one)
   - `habits` (id, user_id, name, created_at, frequency/target)
   - `activity_logs` (id, habit_id, date, completed_at) — one row per completion event
   - `streaks` (derived/cached: current_streak, longest_streak, last_completed_date)
2. **Backend**
   - REST or GraphQL API: CRUD for habits, log activity, fetch streak, fetch calendar data (range query by date)
   - Streak calculation logic (handle timezones, "grace day" rules if any)
3. **Frontend (web)**
   - Habit list + add/edit/delete
   - Streak counter component
   - Contribution calendar component (monthly + annual toggle) — reusable, since v2's public API will need to render the same visual elsewhere
4. **Frontend (app)**
   - Mirror web functionality on mobile shell (React Native / Flutter / native — decide stack)
5. **Testing**
   - Unit tests for streak logic (edge cases: missed day, backfilled entry, timezone rollover)

**Exit criteria:** A user can create habits, log daily activity, and see an accurate streak + calendar on both web and app.

---

## Version 2 — Auth, Dark Mode, Badges, Public API
**Goal:** Multi-user support and a secure, embeddable activity API.

1. **Authentication**
   - Email/password + OAuth (optional) via a standard auth provider or custom JWT-based auth
   - Session/token management, password reset flow
2. **Dark mode**
   - Theming system (CSS variables / design tokens) applied across web + app
3. **Activity badges**
   - Define badge rules (e.g., "7-day streak," "30 activities in a month")
   - Decide: hardcoded rule set (v2) vs. configurable rules engine (defer to later)
   - Badge storage (`badges`, `user_badges` tables) + calculation trigger (on log or via scheduled job)
4. **Secure public API service** *(critical path item)*
   - Scoped API keys per user (read-only, not full account credentials)
   - Endpoints: `GET /public/{user}/activity-graph`, `GET /public/{user}/badges`
   - Rate limiting + key rotation/revocation
   - Return renderable data (JSON) and optionally a hosted SVG/image embed (like GitHub's contribution badge)
   - API docs (OpenAPI/Swagger spec)
5. **Testing/security review**
   - Pen-test the public API scoping (make sure a key can't access anything beyond activity graph/badges)

**Exit criteria:** Multiple users can sign up, use dark mode, earn badges, and generate an API key that a third-party site can use to embed their activity graph.

---

## Version 3 — Android Widget + Admin Dashboard
**Goal:** Extend reach (widget) and give operators visibility (admin analytics).

1. **Lightweight "pending activity" endpoint**
   - Small, cacheable payload optimized for widget refresh (not the full dashboard API)
   - Auth via device-linked token, not full session
2. **Android widget**
   - Home-screen widget showing today's pending habits as a to-do list
   - Tap-to-complete interaction synced back to the main API
   - Background refresh strategy (WorkManager) respecting battery constraints
3. **Web admin dashboard**
   - Build on top of the v2 public API layer where possible (avoid a second internal API)
   - Metrics: total users, active users (DAU/WAU/MAU), habit completion rates, badge distribution
   - Access control: admin-only role, separate from regular user auth

**Exit criteria:** Users can manage habits from an Android widget; admins can view aggregate usage analytics on a dashboard.

---

## Cross-Cutting Concerns (all versions)
- **Timezone handling** — decide once, apply everywhere (streaks, calendar, badges all depend on "what day is it" being consistent)
- **API versioning** — since v2 introduces a *public* API, breaking changes later are costly; version it (`/v1/`, `/v2/`) from the start
- **Observability** — logging/metrics from v1 so v3's admin dashboard has historical data to show, not just data from its launch date

## Suggested Immediate Next Steps
1. Finalize tech stack (backend framework, DB, mobile framework)
2. Design the data model (habits, logs, streaks) in full before writing code
3. Scaffold the repo using the structure in `AGENT.md`
4. Build the streak-calculation logic first — it's the riskiest piece of business logic and everything else depends on it
