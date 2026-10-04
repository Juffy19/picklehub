# PickleHub — Step 4: Application Architecture

> Status: Architecture Definition
> Product source of truth: docs/01-product-requirements.md
> This document defines the application's architectural structure and technical boundaries. It does not replace the product requirements.

---

## 1. Purpose & Scope

This document defines how PickleHub is structured technically: its layers, responsibilities, trust boundaries, consistency model, and the architectural rules that future features must follow.

It covers:

- How responsibilities are divided between presentation, application, domain, data access, and database layers
- Where server/client boundaries lie in the Next.js App Router
- How security, authorization, and data integrity are enforced
- How critical operations remain deterministic and atomic
- How the core product concepts (bookings, Open Play, Playing Sessions, games, results, Smart Rotation, statistics) fit into the architecture
- Which decisions are confirmed, which product questions remain open, and which technical decisions are deliberately deferred

It does not cover:

- Product requirements — these are defined in `docs/01-product-requirements.md`
- Detailed database schema, migrations, or SQL — these belong to the database design stage
- Visual design or the design system
- Implementation code

If this document and the product requirements appear to conflict, the product requirements take precedence and the conflict must be raised rather than silently resolved.

---

## 2. Architectural Principles

1. **Single facility.** PickleHub represents one specific pickleball facility. It is not a multi-facility marketplace. The architecture must not introduce multi-tenant structures.
2. **Deterministic core.** Booking conflicts, availability, capacity, score validation, result confirmation, statistics, session state, and Smart Rotation are deterministic, testable application logic.
3. **Server-authoritative writes.** The browser is never the final authority for sensitive operations. All sensitive writes are decided and performed on the server.
4. **Separation of concerns.** UI, orchestration, business rules, data access, and persistence are kept in distinct layers with explicit dependency rules.
5. **Defense in depth.** Business rules are enforced in TypeScript; database constraints and Row Level Security (RLS) provide a second layer of protection.
6. **Atomic critical operations.** Operations that change several related records (for example, confirming a result and generating the next game) succeed or fail as a whole.
7. **Confirmed data is official data.** Only confirmed results affect official statistics and game progression.
8. **AI is advisory only.** AI never controls core transactional or deterministic systems.
9. **Incremental structure.** The folder structure grows only when real feature requirements justify new boundaries.
10. **No invented rules.** Unresolved product rules are recorded as Open Questions, not decided implicitly in code.

---

## 3. System Architecture

```text
                    ┌──────────────────────────────┐
                    │           Browser            │
                    │  Server-rendered UI + Client │
                    │  Components (interaction)    │
                    └──────────────┬───────────────┘
                                   │ requests / form submissions
                                   ▼
┌──────────────────────────────────────────────────────────────┐
│                   Next.js App (Vercel)                       │
│                                                              │
│  Presentation   App Router pages, layouts, components        │
│        │        Server entry points (thin)                   │
│        ▼                                                     │
│  Application    Use-case orchestration, authorization,       │
│        │        transaction coordination                     │
│        ▼                                                     │
│  Domain         Pure deterministic business rules            │
│                 (no framework or database dependencies)      │
│        │                                                     │
│  Data Access    Server-only database communication, DTOs     │
└────────┼─────────────────────────────────────────────────────┘
         ▼
┌──────────────────────────────────────────────────────────────┐
│                Supabase                                      │
│  Supabase Auth (identity)                                    │
│  PostgreSQL: tables, constraints, RLS, atomic operations     │
└──────────────────────────────────────────────────────────────┘

         Future, advisory only:
         AI Assistant / AI Coach  ──reads authorized data──►  (no control
                                                              of core writes)
```

Technology context (see `CLAUDE.md` and `package.json`):

- Next.js 16 App Router, React 19, TypeScript (strict), Tailwind CSS, React Compiler
- Planned: Supabase (PostgreSQL, Supabase Auth), Vercel deployment
- Next.js behavior is version-sensitive; consult `node_modules/next/dist/docs/` before implementing framework-dependent code (see `AGENTS.md`)

---

## 4. Layered Architecture

The architecture follows:

```text
Presentation → Application → Domain → Data Access → Database
```

The Domain layer is pure: the Application layer calls the Domain layer to make decisions and calls the Data Access layer to load and persist data. The Domain layer itself never calls the Data Access layer.

### 4.1 Presentation Layer

Responsibilities:

- App Router pages, layouts, loading and error states
- Reusable UI components and forms
- Displaying data and collecting user input
- Thin server entry points: Server Components that read data, and Server Functions/Route Handlers that perform actions

Server-side read and write paths:

- Server Components that need protected application data should access it through the server-only Data Access Layer. The Data Access Layer must enforce the appropriate authorization/data-access boundary.
- Not every read needs an application service; a read may go directly through the Data Access Layer when no orchestration is required.
- Sensitive mutations must still go through the Application Layer.

Rules:

- **Server Components are the default.**
- **Client Components are used only when required** by browser interaction, client state, or browser APIs.
- UI must not contain complex business rules.
- UI may use pure domain functions for immediate, non-authoritative feedback (for example, showing that a score looks invalid while typing), but the server always re-validates.
- UI never decides availability, capacity, confirmation status, statistics, or rotation.

### 4.2 Application Layer

Responsibilities:

- Orchestrating use cases such as creating a booking, registering for Open Play, checking in, starting a Playing Session, submitting a result, confirming a result, updating statistics, and generating the next game
- Authenticating the caller and authorizing the specific action on the specific resource
- Validating input at the boundary
- Loading the state required for a decision through the Data Access layer
- Calling the Domain layer to make business decisions
- Persisting decisions atomically through the Data Access layer
- Returning minimal, safe results to the Presentation layer

Rules:

- Application services run only on the server.
- Mutation entry points remain thin and delegate to application services. Protected reads may use the Data Access Layer directly (section 4.1).
- Application services do not contain the business rules themselves; they coordinate them.

### 4.3 Domain Layer

Responsibilities:

- Deterministic business rules and calculations, including:
  - Booking conflict and availability rules
  - Open Play capacity, registration, and check-in rules
  - Score validation under a scoring configuration
  - Result confirmation rules
  - Game and Playing Session rules and state transitions
  - Smart Rotation
  - Statistics calculations

Rules:

- Domain logic is independent of React, Next.js, Supabase, browser APIs, and UI concerns.
- Domain functions receive all inputs explicitly, including the current time when time matters.
- Domain functions do not perform I/O, read environment variables, read the system clock, or use randomness.
- Given the same inputs, a domain function always returns the same output.
- Domain logic is the primary target of automated tests.

### 4.4 Data Access Layer

Responsibilities:

- All communication with Supabase/PostgreSQL
- Mapping database records to typed domain/application data
- Enforcing the authorization/data-access boundary for protected reads requested by Server Components
- Returning safe, minimal data transfer objects (DTOs) for rendering
- Invoking atomic database operations for critical writes

Rules:

- Data access code is server-only and must never be imported into Client Components.
- Database operations are not scattered through UI components.
- Privileged credentials and secret environment variables are read only by server-only modules.
- Data returned for rendering excludes fields the viewer is not permitted to see (for example, private profile information).

### 4.5 Database Layer

Responsibilities:

- PostgreSQL through Supabase
- Durable storage of PickleHub entities (see section 8)
- Database-level integrity: constraints, uniqueness, and referential integrity
- Row Level Security as a second layer of access control
- Atomic persistence of critical multi-record operations

Rules:

- The database protects invariants even if application code has a defect or a request races another request.
- The database does not make product decisions that belong to the Domain layer (see section 7).

### 4.6 Dependency Rules

| Layer | May depend on | Must not depend on |
|---|---|---|
| Presentation (Server Components, entry points) | Application, Data Access (protected reads only), types, pure domain (non-authoritative use) | Data Access for sensitive mutations (these go through Application), database clients directly |
| Presentation (Client Components) | Pure domain (non-authoritative use), types, server entry points passed as actions | Application internals, Data Access, server-only modules, secrets |
| Application | Domain, Data Access, types | Presentation |
| Domain | Types and other pure domain modules only | React, Next.js, Supabase, browser APIs, Data Access, Application, Presentation |
| Data Access | Database client, types | Presentation, Application |
| Database | — | — |

---

## 5. Server / Client Boundaries

1. **Server entry points are public.** Next.js Server Functions are reachable by direct POST requests, not only through the UI. Every Server Function and Route Handler must authenticate and authorize the request itself. A page-level check does not protect the actions used by that page.
2. **Thin entry points.** Mutation entry points parse input, establish the caller's identity, and delegate to an application service. Server Components reading protected data use the server-only Data Access Layer, which enforces the authorization/data-access boundary.
3. **Server-only modules.** Data access, application services, privileged credentials, and AI provider keys are server-only. Next.js supports marking modules as server-only so they cannot be bundled into client code.
4. **Environment variables.** Only variables intentionally prefixed `NEXT_PUBLIC_` reach the browser. Secrets (including any Supabase service-role key and AI API keys) must never use that prefix.
5. **Proxy.** In Next.js 16, Middleware is named Proxy. Proxy may be used for session refresh and optimistic redirects. It is not the authorization layer.
6. **DTOs to the client.** Client Components receive only the minimal data they need, never raw database records.
7. **Shared pure domain code.** Pure domain modules may run on both server and client. Client-side results are hints only; the server's result is authoritative.
8. **Authoritative time.** Server time is authoritative for sensitive time-dependent operations (bookings, check-in, waiting time, result timing). Client clocks are never trusted.
9. **Server-only decisions.** Smart Rotation, result confirmation, statistics updates, availability decisions, and capacity decisions execute on the server.
10. **Freshness.** Availability, capacity, and live session state must not be served from stale caches. Caching strategy is a deferred decision (section 21).
11. **Timezone-consistent rendering.** Dates and times are presented in the configured facility timezone on both server and client.

---

## 6. Security & Authorization

### 6.1 Authentication vs Authorization

- **Authentication** establishes who the caller is (Supabase Auth).
- **Authorization** establishes whether that caller may perform this action on this resource.

These are separate concerns. Being signed in never implies permission to act on a specific booking, session, game, or result.

### 6.2 Roles

| Role | Scope | Notes |
|---|---|---|
| Visitor | Unauthenticated | Can access public content such as the public website. Not a stored role. |
| Player | Global | Authenticated user with a player profile. |
| Session Host | Contextual | Applies to a specific private booking and its Playing Session. Not automatically a global role. The private booking creator is the default Session Host. |
| Facility Admin | Global | Facility management, including resolving disputed results. |
| Facility Staff | Future | May be introduced later. Detailed permissions are not yet defined. |

### 6.3 Authorization Points

- Every server entry point verifies identity and authorization for the specific resource (protecting against insecure direct object references).
- Contextual permissions (for example, Session Host actions) are checked against the specific booking or Playing Session.
- Global permissions (for example, Facility Admin actions) are checked against stored role data, never against client-supplied claims.

### 6.4 Database Security

- RLS is enabled on PickleHub tables as a second protection layer.
- The browser may read data directly only where RLS policies explicitly permit it.
- Sensitive writes (bookings, registrations, check-ins, Playing Sessions, games, results, confirmations, disputes, statistics) are server-authoritative. Direct browser writes to these records are not permitted.
- Database constraints protect invariants independently of application code.
- Privileged database credentials are confined to server-only modules and used as narrowly as possible so that RLS remains meaningful. The exact client/credential strategy is a deferred decision (section 21).

### 6.5 Secrets

- Secrets are never hard-coded and never committed.
- Required variables are documented in `.env.example` without values.

---

## 7. Transaction & Consistency Model

### 7.1 Hybrid TypeScript + PostgreSQL Responsibility Split

| Concern | Owner |
|---|---|
| Business decisions (is this booking allowed, is this score valid, is this result confirmed, who plays next) | TypeScript Domain layer |
| Orchestration, authorization, input validation | TypeScript Application layer |
| Atomic persistence of multi-record changes | PostgreSQL |
| Database-level integrity (constraints, uniqueness, referential integrity) | PostgreSQL |
| Access control second layer | PostgreSQL RLS |

TypeScript owns domain/business decisions. PostgreSQL owns atomic persistence and database-level integrity.

### 7.2 Atomic Critical Operations

Critical operations must be persisted atomically. They include at least:

- Creating a booking (conflict-free for the court and time)
- Registering for Open Play (within capacity)
- Checking in
- Starting a Playing Session and, where the rotation engine is used, generating the first game
- Submitting a result
- Recording a confirmation, and — when confirmation completes — updating any stored derived/cached statistics and generating the next game
- Opening and resolving a dispute
- Adding a late participant to a private Playing Session

The general pattern:

1. The Application layer loads the current state.
2. The Domain layer computes the decision (for example, the next game).
3. The Data Access layer persists the decision in a single atomic database operation.
4. The atomic operation re-verifies that the state the decision was based on is still current (for example, the current game is still confirmed and no next game already exists). If the state changed, the operation fails safely and the application can reload and retry.

### 7.3 Database-Level Invariants

The database must independently protect invariants such as:

- No overlapping confirmed bookings for the same court
- Open Play registrations cannot exceed capacity
- A participant cannot confirm the same result more than once
- A Playing Session cannot have more than one next game generated for the same point in its sequence
- A Playing Session belongs to exactly one origin (a booking or an Open Play session)

Exact constraint implementations are part of database design.

### 7.4 Idempotency and Concurrency

- Repeated or concurrent requests (for example, two players confirming at the same moment) must not create duplicate games, duplicate confirmations, or double-counted statistics.
- Next-game generation must be safe to retry.

### 7.5 Time

- Timestamps are stored with timezone information.
- The facility timezone is configurable facility data. It must not be hardcoded.
- Server time is authoritative for sensitive time-dependent operations.
- Domain functions receive the current time as an explicit input.

---

## 8. Data Architecture

This section describes conceptual data areas only. It is not a schema.

### 8.1 Conceptual Entities

| Area | Conceptual entities |
|---|---|
| Identity | Authenticated users (Supabase Auth), player profiles, role assignments |
| Facility | Facility settings (single facility), including facility timezone and scoring configuration |
| Courts | Courts, court status, operating hours, court blocks/maintenance |
| Bookings | Court bookings, booking participants (authenticated players or guest participants), Session Host |
| Open Play | Recurring Open Play schedules, Open Play sessions, registrations, check-ins |
| Play | Playing Sessions, session participants, games, teams, results, confirmations, disputes |
| Statistics | Official player statistics derived from confirmed results |
| Community | Connections, privacy/visibility settings |
| Future | Events, tournaments, groups, AI interaction data |

### 8.2 Data Rules

- **Single facility.** Facility-level configuration is held as single-facility settings. Tables do not carry multi-facility tenant keys.
- **Identity.** Authenticated identities are managed by Supabase Auth and linked to PickleHub player profiles.
- **Playing Session origin.** A Playing Session originates from exactly one booking or one Open Play session. The model must leave room for a future tournament origin.
- **Historical scoring rules.** Each game stores the scoring rules it was played under (for example, game-to value and whether Win By 2 applies). Later changes to facility scoring configuration must not reinterpret historical games.
- **Recurring Open Play.** The facility can create recurring Open Play schedules that generate individual Open Play sessions; registrations attach to individual sessions.
- **Availability.** Availability is calculated from relevant facility, court, booking, and session state. A stored or client-supplied "available" flag is never trusted as the source of truth.
- **Statistics.** Official statistics are derived from confirmed results; any stored aggregates are rebuildable (section 14).
- **Audit history (architectural recommendation).** Audit history is an architectural recommendation for traceability of sensitive operations and dispute resolution — for example, recording who submitted, confirmed, disputed, or resolved a result, and when. It is not currently defined as a detailed product requirement in Step 1, and its design is not part of this document.

---

## 9. Business Logic Architecture

Pure, deterministic domain areas:

| Domain area | Responsibilities |
|---|---|
| Scoring | Validate a score under a game's scoring configuration; determine the winner |
| Booking | Detect conflicts with confirmed bookings on the same court; compute availability from facility/court/booking/session inputs |
| Open Play | Capacity rules; registration and check-in rules; expansion of recurring schedules into individual sessions |
| Session | Playing Session rules and state transitions; participant eligibility |
| Game & Result | Game state transitions; result submission and confirmation rules; whether the next game may be generated |
| Rotation | Smart Rotation (section 13) |
| Statistics | Calculating official statistics from confirmed results (section 14) |

Rules:

- Business rules live in the Domain layer, not in UI components or database triggers that bypass domain logic.
- Rules that are not yet defined (see section 20) must not be implemented by assumption.
- Configurable rules (for example, scoring configuration) are passed into domain functions as data.

---

## 10. State Lifecycles

Each entity below has an explicit lifecycle enforced by domain logic and protected by database integrity where appropriate. **Status names and the full set of states are not finalized**; they will be defined during database and feature design. The descriptions below capture only what is already established.

### 10.1 Booking

- A booking has a court, date, start time, end time, booking owner, and participants.
- Confirmed bookings for the same court must not conflict. The behavior of non-confirmed booking states remains an open product question.
- A booking can become a Playing Session.
- Open: the full set of booking states (for example, pending, cancelled), cancellation rules, and any payment-related states.

### 10.2 Open Play Session

- Created by the facility, individually or generated from a recurring schedule.
- Has a court, time, and capacity.
- Accepts registrations, then check-ins, and runs as an Open Play Playing Session.
- Open: when registration opens/closes, and how a session is cancelled or ended.

### 10.3 Registration

- A player registers individually for a specific Open Play session.
- Registration is limited by capacity.
- A player can cancel a registration.
- Registration does not mean attendance.

### 10.4 Check-in

- Check-in records that a registered player is present.
- Only checked-in Open Play participants are eligible for rotation.
- Check-in makes a player eligible for rotation; it does not by itself place the player in a game. A checked-in player may still wait (section 11.2).
- Open: who may perform check-in (self, facility staff/admin), the check-in window, and handling of late arrivals or departures.

### 10.5 Playing Session

- Represents actual play on a court.
- Originates from a booking or an Open Play session.
- Is started, runs games, and ends.
- For private bookings, the Session Host can start the session, manage players, start the first game, and end the session.
- Open: exact end conditions (scheduled end time, host/admin action) and handling of a game in progress when a session ends.

### 10.6 Game

- The first game is generated automatically once the session is ready to begin.
- For a session that uses the rotation engine, the host's start-session/start-game action transitions the session into the appropriate ready/active state and triggers deterministic first-game generation. The exact UI action and state transition remain implementation details.
- How the first game is created when rotation is disabled for a private Playing Session, and who starts an Open Play Playing Session, are open questions (section 20).
- A game has teams, a stored scoring configuration, and progresses through play and result recording.
- The next game cannot be generated until the current game has a confirmed result.

### 10.7 Result

- A player submits a result.
- Other participating players can confirm the result.
- A result is not confirmed until the required confirmation process is completed.
- Only confirmed results update official statistics and allow the next game.
- Open: the number of confirmations required, from which participants, and any timeout behavior.

### 10.8 Dispute

- A disputed result can be escalated to a Facility Admin.
- Disputed results do not update official statistics until resolved.
- Open: dispute states, resolution outcomes, and whether a dispute pauses rotation for that session.

---

## 11. Booking vs Open Play Participation Model

Court Booking and Open Play are different products within PickleHub (product requirements §28). Their participation rules are deliberately separate.

| Aspect | Private Court Booking | Open Play |
|---|---|---|
| Created by | Player | Facility |
| Participants | Booking owner and invited/added participants | Individually registered public participants |
| Who manages participants | Session Host, subject to facility/session rules | Registration and check-in rules; no host bypass |
| Guest participants | May be supported (section 12) | Not defined; participation requires registration and check-in |
| Capacity | Booking rules | Session capacity |
| Eligibility for rotation | Participants of the private Playing Session (when rotation is enabled) | Only checked-in participants |
| Rotation | Optional | Automatic |
| Late participants | Can be added and join future games | Subject to registration and check-in rules |

### 11.1 Private Booking Participation

- The private booking creator is the default Session Host.
- The Session Host can manage participants of the private Playing Session, subject to facility/session rules.
- Late participants can join future games but must not alter previous games.
- Smart Rotation is optional for private bookings (it is automatic for Open Play).
- When rotation is enabled for a private Playing Session, the rotation engine organizes games; the host does not manually organize every game.
- Who enables or disables rotation for a private booking, and how games (including the first game) are formed when rotation is disabled, are open questions (section 20).

### 11.2 Open Play Participation

- Open Play is stricter than private bookings.
- Participation requires registration within capacity, followed by check-in.
- Host-style check-in must not bypass registration or capacity rules.
- Registration and attendance/check-in remain separate concepts. Registration does not mean attendance.
- Only checked-in participants are eligible for rotation.

Open Play participation progresses as:

```text
Registered player
→ Check-in
→ Eligible for rotation
→ Selected for a game by the rotation engine
→ Playing
```

- This corresponds to the product requirements' Registered Players → Checked-In Players → Playing Players distinction.
- A checked-in player may still wait and does not automatically participate in the current game.
- Selection for each game is made by the rotation engine; waiting checked-in players remain eligible for future games.

---

## 12. Guest Participant Model

- Private bookings may support guest participants who do not have an authenticated PickleHub account.
- Guest participants are associated with the specific private booking/Playing Session.
- Guest participants must not receive official statistics tied to an authenticated player profile until their identity is safely claimed/verified.
- A guest participant record must never be silently merged into an authenticated profile.
- The guest participant model applies to private bookings. Open Play participation follows the stricter registration and check-in rules (section 11.2); guest participation in Open Play is not defined.

Open questions (section 20): the guest identity claim/verification process; whether guests may submit or confirm results; and whether games involving guests count toward official statistics of the authenticated players in those games.

---

## 13. Smart Rotation Architecture

### 13.1 Principles

- Smart Rotation is deterministic, stable application logic.
- Smart Rotation is never controlled by an LLM or AI model.
- The same input always produces the same output. No randomness.
- It is a reusable module shared by Booking Sessions and Open Play Sessions, and potentially tournaments in the future.
- It runs on the server.

### 13.2 Inputs

The rotation engine receives an explicit snapshot, including:

- Eligible participants (checked-in Open Play participants, or participants of a private Playing Session when rotation is enabled for it)
- Confirmed game history for the session
- Games played and games waited per participant
- Waiting time (computed from server-authoritative time passed in as input)
- Previous partners and previous opponents
- Wins/losses
- Current participant availability
- Format information (initially doubles: four players per game)

### 13.3 Outputs

- The proposed next game: two teams of two, and the waiting participants.

### 13.4 Behavior Established by the Product Requirements

- Supports variable player counts (4 players → 4 play; 5 → 4 play, 1 waits; 6 → 4 play, 2 wait; and so on).
- Prioritizes players who have played fewer games or waited longer.
- Distributes playing and waiting time fairly.
- Avoids repeated partners and unnecessary repeated opponents.

### 13.5 Placement in the Game Flow

- The first game is generated automatically when the session is ready to begin. For a session that uses the rotation engine, the host's start-session/start-game action transitions the session into the appropriate ready/active state and triggers deterministic first-game generation (section 10.6).
- Subsequent games are generated only after the current game's result is confirmed and the confirmed result is reflected in official statistics (section 14).
- The computed next game is persisted atomically with a check that it has not already been generated (section 7).

### 13.6 Not Yet Defined

- Exact factor weights, scoring formula, and tie-breaking rules are open questions (section 20). They must not be invented during implementation.

---

## 14. Statistics Architecture

- **Source of truth.** Confirmed games/results are the source of truth for official statistics.
- **Confirmation first.** Result confirmation must occur before official statistics update.
- **Disputes.** Disputed results do not update official statistics until resolved.
- **Unconfirmed results.** Submitted but unconfirmed results never affect official statistics.
- **Rebuildable.** Any cached or derived statistics must be rebuildable from confirmed results. If a resolved dispute changes an outcome, affected statistics can be recomputed.
- **Persistence.** If statistics are stored as derived/cached values, they must be updated in the same atomic operation as the confirmed result (section 7). If statistics are calculated from confirmed games on read, confirmation of the result is sufficient to make the result part of the statistics source of truth. The stored-vs-calculated choice is a deferred decision (section 21).
- **Calculations.** Statistics calculations are pure domain logic.
- **Statistics in scope** (product requirements §21): games played, wins, losses, win rate, points scored, points against, point differential, Open Play sessions attended, court sessions played, partners played with, opponents played, recent game results.
- **Guests.** Guest participants do not receive official statistics tied to an authenticated profile (section 12).

---

## 15. AI Boundary

AI is advisory only.

### 15.1 AI May

- Provide pickleball education and knowledge assistance
- Provide coaching and strategy explanations
- Provide training recommendations and training plans
- Provide equipment guidance
- Provide progress insights using data the user is authorized to access

### 15.2 AI Must Never Control

- Booking conflicts
- Booking creation
- Capacity
- Registration/check-in authorization
- Score validation
- Result confirmation
- Official statistics
- Session state
- Smart Rotation
- Permissions
- Core transactional writes

### 15.3 Architectural Rules

- AI features access only data the requesting user is authorized to see, through the same server-side authorization as the rest of the application.
- AI output is never treated as an authoritative input to deterministic systems.
- AI provider credentials are server-only.
- AI failure, latency, or unavailability must never prevent core PickleHub operations.

---

## 16. Critical Application Flows

Each flow follows the same pattern: a thin server entry point → an application service that authenticates, authorizes, and validates → pure domain decisions → atomic persistence → a minimal response.

### 16.1 Creating a Court Booking

1. A player selects a date, time, and court in the UI.
2. The server entry point delegates to the booking application service.
3. The service authenticates the player and validates input.
4. It loads relevant facility, court, booking, and session state.
5. The Domain layer determines availability and checks for conflicts with confirmed bookings on the same court.
6. The booking is persisted atomically; the database independently rejects overlapping confirmed bookings.
7. The booking creator becomes the default Session Host.
8. Participants may be invited/added (invitation mechanics are an open question).

### 16.2 Registering for Open Play

1. A player chooses an Open Play session.
2. The service authenticates the player and validates the request.
3. The Domain layer checks registration rules and capacity.
4. The registration is persisted atomically; capacity is protected against concurrent registrations.
5. Registration does not mark the player as present.

### 16.3 Checking In

1. A check-in request is made for a registered participant.
2. The service authorizes the request (who may check in is an open question).
3. The Domain layer verifies the participant is registered for the session and that check-in rules are satisfied.
4. Check-in cannot bypass registration or capacity.
5. The check-in is persisted; the participant becomes eligible for rotation. Eligibility does not mean the participant plays the current game; the rotation engine selects players for each game.

### 16.4 Starting a Playing Session

1. For a private booking, the Session Host starts the session; for Open Play, the session is started under facility rules (exact actor is an open question).
2. The service authorizes the actor for this specific booking or Open Play session.
3. The Domain layer verifies the session can start.
4. The Playing Session is created, linked to exactly one origin.

### 16.5 Automatically Generating the First Game

This flow applies to sessions that use the rotation engine (all Open Play sessions, and private Playing Sessions when rotation is enabled).

1. The start-session/start-game action (by the Session Host for a private booking) transitions the session into the appropriate ready/active state and triggers first-game generation. The exact UI action and state transition remain implementation details.
2. The application service loads eligible participants.
3. Smart Rotation deterministically computes the first game from an empty game history.
4. The game is persisted atomically with its stored scoring configuration.

### 16.6 Submitting and Confirming a Result

1. A player submits the score (which participants may submit is an open question).
2. The Domain layer validates the score against the game's stored scoring configuration.
3. The submitted result is persisted; it is not yet official.
4. Other participating players confirm the result.
5. Each confirmation is persisted once per participant.
6. The Domain layer determines whether the required confirmation process is complete (exact requirement is an open question).

### 16.7 Updating Statistics

1. When the result becomes confirmed, it becomes part of the source of truth for official statistics.
2. If statistics are stored as derived/cached values, they are updated in the same atomic operation as the confirmation that completed the result. If statistics are calculated from confirmed games on read, no separate update is required (the choice is a deferred decision).
3. Disputed or unconfirmed results are excluded.

### 16.8 Running Smart Rotation

1. After the result is confirmed (and any stored statistics are updated), the application service loads the rotation input snapshot (eligible participants, confirmed history, waiting information, server time).
2. The Domain layer computes the next game deterministically.

### 16.9 Generating the Next Game

1. The next game is persisted atomically.
2. The operation verifies that the current game is confirmed and that no next game already exists.
3. Concurrent or repeated requests cannot create duplicate games.

### 16.10 Handling a Disputed Result

1. A submitted result is disputed (who may raise a dispute is an open question).
2. The dispute is recorded (with who raised it and when, per the audit history recommendation in section 8.2).
3. The result does not update official statistics while disputed.
4. The dispute can be escalated to a Facility Admin.
5. The Facility Admin resolves the dispute (resolution outcomes are an open question).
6. Once resolved, the result proceeds according to the resolution; statistics are updated or recomputed as needed.
7. Whether rotation for the session pauses during a dispute is an open question. The rule that the next game is not generated until the current game has a confirmed result is not bypassed.

### 16.11 Adding a Late Participant to a Private Booking

1. The Session Host adds a participant (authenticated player or guest) to the private Playing Session.
2. The service authorizes the host for this specific session and applies facility/session rules.
3. The participant becomes eligible for future games.
4. Previous games, results, confirmations, and statistics are not altered.
5. How rotation fairness accounts for a late participant is part of the open rotation-rules question.

---

## 17. Testing Strategy

No test framework is currently installed. Selecting and adding one is a deferred decision (section 21) to be made before the first domain module is implemented.

| Level | Purpose | Examples |
|---|---|---|
| Domain unit tests | Thoroughly verify pure deterministic rules | Score validation including Win By 2 on/off and boundaries; booking overlap and availability; capacity; confirmation rules; state transitions; rotation determinism and fairness for variable player counts; statistics calculations |
| Database constraint/RLS tests | Verify the database independently protects invariants and access | Overlapping confirmed bookings rejected; capacity enforced; duplicate confirmations rejected; duplicate next games rejected; RLS denies unauthorized reads/writes |
| Application flow/integration tests | Verify orchestration, authorization, and atomicity | Confirm → (stored statistics, if used) → next game as one unit; concurrent confirmations; unauthorized actors rejected; disputes block official statistics |
| End-to-end tests (where appropriate) | Verify critical user journeys in the running application | Booking a court; registering and checking in for Open Play; playing through a session |

Rules:

- Deterministic business rules require thorough tests.
- A successful build alone does not mean a feature is complete.
- Linting and type checking run alongside tests.

---

## 18. Project Structure & Conventions

The current structure is intentionally small:

```text
src/
├── app/
├── components/
├── lib/
└── types/
```

- `src/app/` — App Router routes, layouts, and server entry points
- `src/components/` — reusable UI components
- `src/lib/` — shared logic and utilities
- `src/types/` — shared TypeScript types

Growth conventions, applied only when actual feature requirements justify them:

- Pure domain modules may be introduced under `src/lib/domain/<area>`.
- Server-only modules (application services, data access) may be introduced under `src/lib/server/...`.

Conventions:

- Do not create folders or abstractions in advance of real requirements.
- Pure domain modules must not import from `src/app/`, `src/components/`, `src/lib/server/`, React, Next.js, or Supabase.
- Server-only modules must be marked and kept out of client bundles.
- Use the `@/*` path alias for imports from `src/`.
- Follow `CLAUDE.md` for workflow, code quality, and Git rules.

---

## 19. Architecture Decision Log

All decisions in this table are **Confirmed**.

| ID | Decision | Rationale |
|---|---|---|
| ADR-01 | **Single-facility model.** PickleHub represents one specific facility; no multi-facility tenancy. | The product is a digital home for one facility, not a marketplace. Avoiding tenancy keeps the model and authorization simpler. |
| ADR-02 | **Layered architecture:** Presentation → Application → Domain → Data Access → Database. | Separates UI, orchestration, business rules, and persistence so each can be understood and tested independently. |
| ADR-03 | **Deterministic domain logic** independent of React, Next.js, Supabase, browser APIs, and UI. | Core rules (booking, scoring, confirmation, rotation, statistics) must be testable and produce the same output for the same input. |
| ADR-04 | **Server-authoritative writes.** The browser is never the final authority; server entry points are thin and authorize every request. | Server Functions are publicly reachable; client state can be manipulated. Sensitive decisions must be made and verified on the server. |
| ADR-05 | **Hybrid TypeScript + PostgreSQL transaction architecture.** TypeScript owns business decisions; PostgreSQL owns atomic persistence and integrity, with RLS and constraints as a second protection layer. | Keeps rules in testable TypeScript while guaranteeing atomicity and protecting invariants against races and application defects. |
| ADR-06 | **Confirmed results are the source of truth for official statistics;** derived statistics are rebuildable. | Prevents unconfirmed or disputed results from affecting statistics and allows recomputation after dispute resolution. |
| ADR-07 | **Configurable facility timezone;** server time is authoritative. | Hosting runs in UTC while facility schedules are local; hardcoding a timezone or trusting client clocks would produce incorrect bookings and schedules. |
| ADR-08 | **Historical scoring configuration stored per game.** | Facility scoring configuration can change (game-to value, Win By 2); past games must be validated and interpreted by the rules they were played under. |
| ADR-09 | **Private booking guest support** without official statistics until identity is safely claimed/verified. | Real private groups include people without accounts, but official statistics must remain trustworthy and tied to verified identities. |
| ADR-10 | **Stricter Open Play participation rules:** registration within capacity, then check-in; no host bypass. | Open Play is a public, facility-run product with capacity limits; participation must be fair and verifiable. |
| ADR-11 | **Advisory-only AI.** AI never controls core transactional or deterministic systems, and AI failure never blocks core operations. | Core rules must remain deterministic, auditable, and available. |
| ADR-12 | **Server Components by default;** Client Components only when browser interaction, state, or APIs require them. | Keeps data access and secrets on the server and minimizes client-side code. |
| ADR-13 | **Authentication and authorization are separate;** roles initially Player, Session Host (contextual), and Facility Admin. | Prevents "signed in" from being mistaken for "permitted"; Session Host authority is scoped to a specific private booking. |
| ADR-14 | **Availability is calculated** from facility/court/booking/session state, never from a trusted client-side flag. | Availability is a derived, deterministic result and a common target for manipulation. |
| ADR-15 | **Small, incremental project structure.** `src/lib/domain/<area>` and `src/lib/server/...` introduced only when justified. | Avoids premature abstraction while providing an agreed path for growth. |

---

## 20. Open Questions

These are **open product questions**. They must be answered before the related feature is implemented. They must not be resolved by assumption in code.

### 20.1 Results & Disputes

1. How many confirmations are required for a result to be confirmed, and from which participants (for example, an opponent, any other participant)?
2. What happens if a result is not confirmed within a reasonable time? Is there a timeout?
3. Which participants may submit a result (for example, any participant, the Session Host, guest participants)?
4. Who may raise a dispute, what are the dispute states, and what are the possible resolution outcomes?
5. Does a dispute pause rotation for the session, and how does play continue while it is unresolved?

### 20.2 Smart Rotation

6. What are the exact factor weights, scoring formula, and tie-breaking rules?
7. How are wins/losses used in rotation decisions?
8. Can a single Open Play session use more than one court?
9. Is singles play supported, or only doubles initially?
10. How does rotation treat participants who join late or leave mid-session?
11. For private bookings, where rotation is optional: who enables or disables rotation, what happens when rotation is disabled, and how is the first game created when rotation is disabled?

### 20.3 Bookings

12. What booking states exist (for example, pending, confirmed, cancelled), how does a booking become confirmed, and how do non-confirmed booking states affect availability and conflicts?
13. What are the booking time-slot granularity and minimum/maximum durations?
14. Are back-to-back bookings (one ending exactly when the next begins) allowed?
15. What are the cancellation rules and the advance-booking window?
16. How do Open Play sessions and court blocks/maintenance interact with booking availability?
17. Are there participant limits for private bookings?

### 20.4 Open Play

18. Who may perform check-in (self check-in, Facility Admin, future Facility Staff)?
19. What is the check-in window, and how are late arrivals handled?
20. Who starts and ends an Open Play Playing Session (and thereby triggers first-game generation)?
21. Are waitlists supported when capacity is reached?

### 20.5 Participants & Guests

22. How are participants invited to a private booking, and must invitations be accepted?
23. What is the process for a guest to claim/verify their identity?
24. May guest participants submit or confirm results?
25. Do games involving guest participants count toward the official statistics of the authenticated players in those games?

### 20.6 Payments, Roles & Other

26. Are fees (booking fee, Open Play fee) collected online? Does payment affect booking or registration states?
27. What permissions will Facility Staff have, if introduced?
28. What is the default profile visibility, and which fields can players control?
29. What counts as "attending" an Open Play session or "playing" a court session for statistics (check-in, or playing at least one game)?
30. How is a game in progress handled when a Playing Session ends?
31. Is Learn content managed in the codebase or by facility administrators?

---

## 21. Deferred Decisions

These are **deferred technical decisions**. They do not require new product rules but will be decided at the appropriate implementation stage.

| Decision | Decide by |
|---|---|
| Test framework selection and setup | Before the first domain module |
| Input validation approach/library | Before the first server entry point that accepts input |
| Exact atomic persistence mechanism per operation (for example, database functions invoked from the server) | Database design |
| Supabase client and credential strategy (user-scoped vs privileged access) | Database/auth design |
| Migration tooling and generated database types | Database design |
| Exact constraint implementations for invariants | Database design |
| Mechanism for generating Open Play sessions from recurring schedules | Open Play implementation |
| Statistics storage strategy (computed on read vs stored rebuildable aggregates) | Statistics implementation |
| Next.js caching strategy (including whether to enable Cache Components) | When the first data-driven pages are built |
| Live session updates (realtime, polling, or refresh) | Playing Session implementation |
| Notification/invitation delivery mechanism | Booking invitations implementation |
| Error monitoring and observability | Before production deployment |
| Rate limiting for public entry points | Before production deployment |
| AI provider, model, and data-access design | AI feature stage |
| Expansion of the project structure beyond the current folders | When feature requirements justify it |

---

## 22. Current Development Stage

Completed:

- Product definition (`docs/01-product-requirements.md`)
- Development environment and Next.js project initialization
- Claude Code configuration (`CLAUDE.md`)
- Application architecture definition (this document)

Current implementation state:

- `src/app/` contains the initial Next.js scaffold
- `src/components/`, `src/lib/`, and `src/types/` exist locally but contain no code yet
- No database, authentication, or test framework is configured yet

Next stage:

- Database & implementation design, guided by the confirmed decisions in section 19
- Open questions in section 20 are resolved as the related features are reached

---

> Status: READY FOR DATABASE & IMPLEMENTATION DESIGN
