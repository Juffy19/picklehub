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

The detailed product specification is maintained separately in the project's Step 1 documentation.

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

---

## 4. Important Product Rules

### Court Booking

- Bookings must not conflict with confirmed bookings for the same court.
- A booking contains a date, start time, end time, court, and booking owner.
- Multiple players may participate in a booking.
- A booking can become a Playing Session.
- Private court bookings have a Session Host.

### Open Play

Open Play is a scheduled public session created by the facility.

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

### Game Results

The default scoring configuration is:

- Game to 11
- Win by 2

Examples:

Valid:
- 11–8
- 11–9
- 12–10

Invalid:
- 11–10

Game scoring should eventually support facility configuration such as games to 11, 15, or 21.

### Result Confirmation

A player may submit a game result.

Other participating players can confirm the result.

A result is not considered confirmed until the required confirmation process is completed.

Disputed results should be capable of being escalated to a facility administrator.

### Critical Game Flow Rule

Never generate the next game until the current game's result has been confirmed.

The sequence is:

Play Game
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

Preferred workflow:

Define
→ Implement
→ Test
→ Review
→ Commit
→ Push

Do not attempt to implement the entire platform in one change.

For each feature:

1. Understand the existing architecture.
2. Identify affected files.
3. Make the smallest appropriate change.
4. Run relevant tests/checks.
5. Run linting.
6. Verify the application.
7. Review the diff.
8. Commit the completed change.

Do not modify unrelated features.

---

## 9. Git Workflow

Use the `main` branch.

Use descriptive conventional-style commit messages.

Examples:

- `feat: add court availability page`
- `feat: add open play registration`
- `feat: add game result confirmation`
- `fix: prevent conflicting court bookings`
- `refactor: extract rotation engine`
- `docs: update project documentation`

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

## 13. Project Documentation

The Step 1 product specification is the source of truth for product requirements.

When implementing a feature, follow the established product rules before introducing new behavior.

If a requirement is ambiguous or conflicts with an existing documented rule, stop and ask for clarification rather than silently inventing behavior.

---

## 14. Current Development Stage

The project is currently in the initial technical setup stage.

Completed:

- Product definition
- Development environment
- Next.js project initialization
- Git repository
- GitHub repository

Upcoming stages include:

- Project architecture
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