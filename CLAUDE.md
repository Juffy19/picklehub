@AGENTS.md

# PickleHub — Claude Code Instructions

## 1. Project Overview

PickleHub is a modern web platform for a specific pickleball facility.

It is not a marketplace for discovering pickleball courts in different locations.

The platform provides a digital home for players to:

- Learn pickleball
- View the facility's courts
- Book courts
- Join Open Play sessions
- Participate in events and tournaments
- Create player profiles
- Record games
- Track wins, losses, and statistics
- Connect with other players
- Eventually use AI-powered pickleball assistance and coaching

Product requirements:
`docs/01-product-requirements.md`

This finalized Step 1 product specification is the primary source of product requirements. See section 13 for how project documents relate to each other.

---

## 2. Technology Stack

Current stack:

- Next.js 16
- React
- TypeScript
- Tailwind CSS
- ESLint
- React Compiler
- Next.js App Router
- Git / GitHub
- Vercel for deployment

Planned backend:

- Supabase
- PostgreSQL
- Supabase Auth

Planned AI capabilities:

- AI Pickleball Assistant
- AI Coach
- AI-generated training plans

Do not add technologies or dependencies without a clear project requirement.

---

## 3. Core Architecture Principles

Keep UI, business logic, data access, and reusable utilities appropriately separated.

Prefer:

- Small reusable components
- Typed data structures
- Clear domain logic
- Server-side logic where appropriate
- Reusable services/utilities
- Explicit validation
- Maintainable code over clever code

Do not build the entire application in a single component or file.

Do not introduce unnecessary abstractions before they are needed.

Architecture rules:

- Keep domain logic pure, typed, deterministic, and testable where practical.
- Keep domain/business logic separate from UI concerns. Do not scatter business rules throughout UI components.
- Do not make significant architectural decisions (decisions that could affect future features) without first explaining the trade-offs and getting agreement.
- Do not replace working architecture simply because another approach is possible.
- Do not add a dependency without a clear technical need.
- For version-sensitive Next.js behavior, consult the version-matched documentation in `node_modules/next/dist/docs/` before implementing (see `AGENTS.md`).

---

## 4. Important Product Rules

### Non-Negotiable Business Rules

- PickleHub represents one specific pickleball facility.
- It is not a marketplace for discovering courts across multiple locations.
- Booking availability and booking conflicts are deterministic.
- Score validation is deterministic.
- Result confirmation is deterministic.
- Official statistics use only confirmed results.
- Smart Rotation is deterministic application logic and must never be controlled by an LLM.
- The next game must never be generated before the current game's result is confirmed.
- AI is advisory only and may assist with pickleball knowledge, coaching, recommendations, training plans, and progress insights.
- AI must not control booking availability, booking conflicts, booking creation, score validation, result confirmation, statistics calculation, or Smart Rotation.
- Never silently change an established business rule.
- Never invent a missing business rule. Ask when the requirement is ambiguous.

### Court Booking

- Bookings must not conflict with confirmed bookings for the same court.
- A booking contains a date, start time, end time, court, and booking owner.
- Multiple players may participate in a booking.
- A booking can become a Playing Session.
- Private court bookings have a Session Host.

### Open Play

Open Play is a scheduled public session created by the facility.

Each Open Play session has a defined capacity.

The facility can create recurring Open Play schedules (for example, every Saturday 1:00 PM–4:00 PM) that generate individual Open Play sessions.

Step 5 decisions (see `docs/03-step-5-database-decisions.md`):

- Recurring Open Play uses a Recurring Open Play Schedule (MVP: weekly by day of week, start/end time, start date, optional end date, facility timezone). Each generated instance is independent; schedule changes never rewrite existing instances. Generated instances are validated against operating hours, court availability, bookings, blockouts, and closures; conflicts require Facility Admin review and are never silently resolved.
- An Open Play may reserve one or more courts.
- Capacity is registration capacity. No waitlist in MVP.
- Open Play has an informational skill level (BEGINNER, INTERMEDIATE, ADVANCED, ALL_LEVELS) that does not restrict registration, check-in, or rotation.
- Open Play is started explicitly by the designated Open Play Host, who must be a FACILITY_ADMIN. It never starts automatically at the scheduled time.
- Players may self check-in; a FACILITY_ADMIN may correct attendance.
- Check-In (CHECKED_IN / CHECKED_OUT) is attendance only. Session Participant status (ACTIVE / LEFT) is the sole authority for future gameplay eligibility; checking out makes the Session Participant LEFT and does not end an active game.
- With fewer than 4 eligible checked-in players at SESSION_READY, no Playing Session or Games are created and the Open Play may be COMPLETED. CANCELLED is a separate terminal state for facility cancellation before activation.
- No anonymous guests in Open Play in MVP. Open Play fees are informational only.

Players register individually.

Players do not need to select a partner when registering.

Registration and attendance are separate concepts:

Registered Players
→ Checked-In Players
→ Playing Players

Only checked-in players participate in game rotation.

### Playing Sessions

A Playing Session represents actual play occurring on a court.

Initially supported session types include:

- Booking Session
- Open Play Session

The game and rotation systems should be reusable across these session types.

The first game in a Playing Session is generated automatically once the session is ready to begin.

Smart Rotation is automatic for Open Play sessions. For normal/private court bookings, rotation may be optional.

Step 5: a private Playing Session selects a gameplay mode before gameplay begins — SMART_ROTATION or FIXED_PARTNERS in MVP; TOURNAMENT is a recognized future mode that is not enabled in MVP. The mode must not silently change after gameplay begins. FIXED_PARTNERS uses persistent Session Teams and deterministic team-vs-team matchup and court generation (not Smart Rotation). All modes reuse the Game → Result → Confirmation → Statistics pipeline.

Private bookings: the Booking Owner automatically participates; other registered players join through accepted invitations; guests are added directly by the owner. Booking end is a hard boundary — no new Game starts after it. Payment processing is excluded from MVP.

### Session Host

For private court bookings, one player is designated as the Session Host.

The Session Host can:

- Start the session
- Manage players
- Start the first game
- End the session
- Handle basic session management

The Session Host does not need to manually organize every game. The rotation engine organizes subsequent games according to the established rules.

Step 5 contextual roles (none of these is a global role):

- **Open Play Host** — designated per Open Play; must be an authorized FACILITY_ADMIN; has normal Session Host gameplay authority for that Open Play's Playing Session.
- **Authorized Scorer** — designated per Playing Session by the Session Host or a FACILITY_ADMIN; must be a registered user; may submit final scores; cannot confirm Game completion in MVP.

Global roles are PLAYER and FACILITY_ADMIN; one account may hold both. Facility Staff is deferred.

### Game Results

The default scoring configuration is:

- Game to 11
- Win by 2

Examples (with Win By 2 enabled):

Valid:
- 11–8
- 11–9
- 12–10

Invalid:
- 11–10

Game scoring should eventually support facility configuration of:

- Game To: 11, 15, or 21
- Win By 2: Yes / No

The examples above apply when Win By 2 is enabled.

### Result Confirmation

*Updated by Step 5 (U1–U5, U15–U17, B1, I1, I8). The earlier rule "Other participating players can confirm the result" and the open question about the number of confirmations are superseded.*

- The Session Host or an Authorized Scorer submits final team scores (final score only). A FACILITY_ADMIN may also submit.
- The Session Host confirms "Game Complete", including for a game they played in. A FACILITY_ADMIN may confirm when necessary. Authorized Scorers cannot confirm in MVP. Player confirmation is not required.
- A result is not confirmed until the Session Host (or a FACILITY_ADMIN) confirms it. There is no automatic timeout or automatic confirmation.
- The winner is derived from the validated score; invalid scores are rejected.
- Game Results are versioned and never overwritten. Every resubmission or correction creates a new version, and the previous version becomes SUPERSEDED (including while still SUBMITTED). Exactly one CONFIRMED version is authoritative.
- Lifecycles are separate: Game SCHEDULED → ACTIVE → COMPLETED; Game Result SUBMITTED → CONFIRMED or SUPERSEDED; Dispute DISPUTED → ADMIN_REVIEW → RESOLVED.
- Registered participants may dispute a confirmed result (the Session Host may file on a guest's behalf). A dispute does not reopen the Game, and later games are not deleted or regenerated.
- A FACILITY_ADMIN resolves disputes, but not when that admin is the Session Host, the Open Play Host, or a participant in the disputed Game. If no eligible admin is available, the dispute remains unresolved.
- Wrong player/team assignments are corrected non-destructively by a FACILITY_ADMIN.

### Critical Game Flow Rule

The first game is generated automatically.

Never generate the next game until the current game's result has been confirmed.

The sequence is:

Generate First Game
→ Play Game
→ Submit Result
→ Confirm Result
→ Update Statistics
→ Generate Next Rotation
→ Generate Next Game

Do not bypass this sequence.

---

## 5. Smart Rotation Rules

Smart Rotation is deterministic application logic.

Do NOT use an LLM or AI model to determine the core game rotation.

Rotation is automatic for Open Play sessions and may be optional for normal/private court bookings.

The rotation system should consider:

1. Previous partners
2. Previous opponents
3. Number of games played
4. Number of games waited
5. Waiting time
6. Wins/losses
7. Current player availability

Primary goals:

- Fair distribution of playing time
- Fair distribution of waiting time
- Variety of partners
- Variety of opponents

Players with fewer games or longer waiting times should receive appropriate priority.

The system must support variable player counts.

Examples:

- 4 players → 4 play
- 5 players → 4 play, 1 waits
- 6 players → 4 play, 2 wait
- 7 players → 4 play, 3 wait
- 8 players → 4 play, 4 wait

Do not replace deterministic rotation logic with AI-generated decisions.

---

## 6. Player Statistics

Confirmed games should update player statistics.

Relevant statistics include:

- Games played
- Wins
- Losses
- Win rate
- Points scored
- Points against
- Point differential
- Open Play sessions attended
- Court sessions played
- Partners played with
- Opponents played
- Recent game results

Only confirmed results should update official player statistics.

Step 5: official statistics use the authoritative CONFIRMED Game Result that is not under an active Dispute, together with the authoritative Game participation (including Facility Admin corrections). Cached statistics are optional, never authoritative, and must be rebuildable. Guests do not receive persistent global statistics.

---

## 7. AI Boundaries

AI is a future feature of PickleHub.

AI may be used for:

- Pickleball knowledge assistance
- Beginner guidance
- Strategy explanations
- Practice recommendations
- Training plans
- Personalized coaching insights
- Progress summaries

AI should NOT control core transactional or deterministic systems such as:

- Court availability
- Booking conflicts
- Booking creation
- Game score validation
- Result confirmation
- Player statistics calculations
- Smart Rotation

Core application rules must remain deterministic and testable.

---

## 8. Development Workflow

Build PickleHub incrementally.

Do not attempt to implement the entire platform in one change.

Follow this workflow for every change:

Understand
→ Plan
→ Implement
→ Test
→ Review
→ Commit
→ Push

### Understand

- Read affected files before changing them.
- Inspect related types, utilities, tests, and documentation.
- Understand the current architecture.
- Refer to `docs/01-product-requirements.md` for product behavior.
- Do not invent requirements.

### Plan

- For non-trivial work, identify the approach, affected files, business rules, edge cases, and scope.
- Keep the work scoped to the requested feature.
- Ask before implementing ambiguous or architecture-significant decisions.

### Implement

- Make the smallest clean change that satisfies the requirement.
- Follow the existing architecture and coding conventions.
- Do not modify unrelated files or features.
- Do not add unnecessary dependencies.
- Keep domain/business logic separate from UI concerns.
- Prefer typed, reusable, testable code.

### Test

- Run appropriate tests.
- Run linting and type checking where applicable.
- Deterministic business rules must have thorough tests.
- Verify the application where the change affects it.
- A successful build alone does not mean a feature is complete.

### Review

- Review `git status` and `git diff`.
- Check for regressions, unused code, security problems, leaked secrets, unrelated changes, and business-rule violations.
- Report failures, assumptions, skipped checks, and remaining limitations honestly.

### Commit

- Commit only after implementation and verification are complete.
- Use conventional commit messages (see section 9).
- Never commit secrets, `.env.local`, `node_modules`, or `.next`.

### Push

- Push only after the commit has been reviewed and verified.
- Keep `main` in a working state.

---

## 9. Git Workflow

Use the `main` branch.

Use descriptive conventional-style commit messages with prefixes such as `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, and `chore:`.

Examples:

- `feat: add court availability page`
- `feat: add open play registration`
- `feat: add game result confirmation`
- `fix: prevent conflicting court bookings`
- `refactor: extract rotation engine`
- `test: cover score validation edge cases`
- `docs: update project documentation`
- `chore: update tooling configuration`

Do not commit:

- `.env.local`
- API keys
- secrets
- credentials
- `node_modules`
- `.next`

Always inspect changes before committing.

---

## 10. Environment Variables

Never hard-code secrets.

Use environment variables for:

- Supabase configuration
- Authentication configuration
- AI API keys
- Other external services

Use `.env.example` to document required variables without exposing secret values.

---

## 11. Code Quality

Prefer:

- TypeScript types over `any`
- Explicit domain types
- Reusable components
- Accessible UI
- Responsive layouts
- Clear naming
- Small focused functions
- Testable business logic

Avoid:

- Unnecessary dependencies
- Large monolithic components
- Duplicated business logic
- Hard-coded business rules scattered throughout UI
- AI-generated logic that bypasses established product rules

---

## 12. Before Changing Existing Code

Before modifying an existing feature:

1. Read the relevant files.
2. Understand how the current implementation works.
3. Check related types and utilities.
4. Check whether the behavior is covered by existing tests.
5. Preserve existing functionality unless the task explicitly changes it.

Do not rewrite working systems unnecessarily.

---

## 13. Project Documentation and Source of Truth

Product requirements:
`docs/01-product-requirements.md`

Document roles:

- `docs/01-product-requirements.md` — the finalized Step 1 product specification and the primary source of product requirements (including its Post-Step-1 Amendments).
- `docs/02-application-architecture.md` — the application architecture and technical boundaries.
- `docs/03-step-5-database-decisions.md` — the approved Step 5 database and implementation decisions, including resolved decisions U1–U19 and the final consistency decisions. It is the source of truth for entity, lifecycle, and data-integrity decisions before physical schema design.
- `CLAUDE.md` — Claude Code's development behavior and project-specific engineering rules.
- `README.md` — human-facing project documentation.
- `AGENTS.md` — framework and agent guidance (including Next.js version notes).

When implementing a feature, follow the established product rules before introducing new behavior.

If these documents appear to conflict, do not silently choose a requirement. Explain the conflict and ask before changing established product behavior.

If a requirement is ambiguous or missing, stop and ask for clarification rather than silently inventing behavior.

---

## 14. Current Development Stage

The next stage is Step 6 — Physical Database Schema Design.

Completed:

- Product definition
- Development environment
- Next.js project initialization
- Git repository
- GitHub repository
- Application architecture (`docs/02-application-architecture.md`)
- Step 5 Database & Implementation Decisions — finalized (`docs/03-step-5-database-decisions.md`)

Upcoming stages include:

- Step 6 — Physical Database Schema Design
- Design system
- Public website
- Court pages
- Court availability
- Booking system
- Open Play
- Accounts
- Playing Sessions
- Game recording
- Smart Rotation
- Player statistics
- Community
- Admin dashboard
- AI features