# PickleHub — Step 5: Database & Implementation Decisions

> Status: Decision Record
> Product source of truth: `docs/01-product-requirements.md`
> Application architecture: `docs/02-application-architecture.md`
> This document records the business rules, entity concepts, relationships, lifecycle states, integrity rules, and database-shaping decisions locked during Step 5. It is not a physical database schema.

---

## 1. Purpose

Step 5 was used to decide the business rules, entity concepts, relationships, lifecycle states, integrity rules, and database-shaping decisions required before physical database schema design.

The process was:

```text
Decide → Document → Review → Commit → Push → Implement
```

No SQL/schema implementation should happen until the conceptual decisions are verified.

This document:

- Is the finalized Step 5 database decision record, including the resolved decisions in section 10
- Has been verified through consistency audits against `docs/01-product-requirements.md` and `docs/02-application-architecture.md`
- Is the input and basis for Step 6 — Physical Database Schema Design

This document does not:

- Contain SQL, migrations, or a Supabase schema
- Define physical data types, indexes, constraints, triggers, functions, or RLS policies (these are Step 6 decisions)

---

## 2. How to Read This Document

### 2.1 Decision Status

| Marker | Meaning |
|---|---|
| **LOCKED** | Decided during Step 5. Must not be changed, simplified, or reinterpreted without an explicit new decision. |
| **DEFERRED** | Intentionally not decided. Must not be turned into an invented requirement during implementation. |

All decisions in section 3 are **LOCKED** unless an individual item is explicitly marked **(DEFERRED)**. Section 8 consolidates all deferred decisions.

### 2.1.1 Later Resolutions

Section 3 preserves the original Step 5 record. The Step 5 consistency audits produced additional **approved** decisions (U1–U19, B1, B2, I1, I3, I4(c), I5/I6, I7, I8, I12, I14), recorded in section 10.

- Where a section 3 rule has been refined or superseded, the affected subsection carries an **"Updated by …"** note.
- **Section 10 takes precedence** over section 3 wherever they differ.
- Section 11 maps every supersession. Section 12 lists remaining non-blocking items.

### 2.2 Conceptual Fields

Field lists in this document are **conceptual**. They identify the information an entity holds. Physical data types, units, nullability, keys, constraints, and indexes are schema-design decisions (DEFERRED).

### 2.3 Terminology

Terminology is defined in section 9 and must be used consistently. Smart Rotation must never be called "Random".

---

## 3. Locked Decisions

### 5.1 Booking Lifecycle

**Booking states:**

```text
CONFIRMED
CANCELLED
COMPLETED
```

**Rules:**

- Booking is confirmed immediately after successful server validation.
- No pending approval state for MVP.
- Cancellation changes status rather than deleting the booking.
- Completed means the scheduled booking period/session has completed.
- Booking uses `start_time` and `end_time`.
- There are no fixed predefined time slots.
- Back-to-back bookings are allowed.
- Time interval model is `[start_time, end_time)`, meaning start is inclusive and end is exclusive.
- Advance booking window is configurable.
- Booking duration is configurable.
- Booking hours are configurable.
- Historical booking records are retained.
- Exact configuration values belong to Facility Configuration.

---

### 5.2 Court Availability & Conflict Rules

**Rules:**

- Availability is calculated rather than stored as a simple boolean.
- A confirmed booking makes a court unavailable for overlapping time.
- Open Play reserves its selected court(s).
- Multiple courts are supported.
- An Open Play may reserve one or more courts.
- Court status:
  - `ACTIVE`
  - `INACTIVE`
- Temporary closure is represented by a court blockout.
- Permanent/current unavailability is represented by court status.
- Facility-wide closure is supported separately.
- Operating hours are configurable.
- Server + database are the final authority.
- UI availability is informational only.
- Conflict interval is `[start,end)`.
- Facility hours are the default for courts.
- Individual court operating hours are **(DEFERRED)**.
- Historical data is retained.

**Overlap rule:**

```text
requestedStart < existingEnd
AND
requestedEnd > existingStart
```

**Conceptually, court availability requires:**

```text
active court
AND within operating hours
AND no booking overlap
AND no Open Play overlap
AND no blockout overlap
AND no facility closure overlap
```

---

### 5.3 Open Play Lifecycle

> **Updated by U13, I4(c), U19, U8, U9 (see section 10).** CANCELLED is an explicit terminal state (U13). An Open Play reaching SESSION_READY with fewer than 4 eligible checked-in players may transition to COMPLETED without a Playing Session (I4(c)). "Facility admin/staff" and "designated Open Play host" mean FACILITY_ADMIN in MVP (U19, U14). Recurring instances are generated from a Recurring Open Play Schedule (U8, B2). Open Play has an informational skill level (U9).

**Open Play lifecycle:**

```text
SCHEDULED
→ REGISTRATION_OPEN
→ SESSION_READY
→ ACTIVE
→ COMPLETED
```

Cancellation is also supported.

**Rules:**

- Open Play is created by the facility.
- Players register individually.
- Partner registration is not required.
- Capacity is enforced.
- There is NO waitlist in MVP.
- Registration, check-in, and playing are separate concepts.
- A player normally must be registered before checking in.
- Checked-in players are eligible for rotation.
- Player self-check-in is supported.
- Facility admin/staff can correct attendance.
- Late arrivals can join future games, but do not interrupt the current game.
- Early leavers are excluded from future rotation.
- Historical games remain preserved.
- Open Play is started explicitly by:
  - facility admin/staff, OR
  - designated Open Play host.
- Open Play does NOT automatically start merely because the scheduled clock time is reached.
- Session starts only when explicitly started and enough eligible checked-in players exist.
- Minimum doubles players = 4.
- 4 players → 4 play.
- 5 players → 4 play + 1 waits.
- 6 players → 4 play + 2 wait.
- Zero checked-in players may result in an Open Play ending without games.
- Session ending is explicit.
- Capacity refers to registration capacity, not concurrent on-court players.
- No anonymous Open Play guests in MVP.
- Open Play may reserve multiple courts.
- Recurring Open Play should create separate Open Play instances.
- Each recurring instance has its own registrations, attendance, gameplay, results, and history.

---

### 5.4 Playing Sessions & Games

> **Updated by U2–U4, U5, U6, U7, U17, U18, U19, I8 (see section 10).** Game states are SCHEDULED → ACTIVE → COMPLETED (U2–U4). TOURNAMENT is a future mode and origin, not enabled in MVP (U7). FIXED_PARTNERS first-game and matchup generation is defined by U6. Booking end is a hard boundary (U18). The Open Play starter is a FACILITY_ADMIN acting as Open Play Host (U19). There is no automatic result timeout (I8).

Playing Session is the container for actual gameplay.

**Session origins:**

```text
BOOKING
OPEN_PLAY
TOURNAMENT (future)
```

**Session states:**

```text
CREATED
READY
ACTIVE
COMPLETED
```

**Rules:**

- Private booking session starter = Session Host.
- Open Play starter = facility admin/staff/designated Open Play host.
- Session participants are distinct from currently playing players.
- Games belong to Playing Sessions.
- There may be multiple courts in a session.
- Correct active-game rule: maximum one ACTIVE game per court within a session.
- A player cannot participate in two simultaneous active games within the same session.
- MVP scoring is Game to 11, Win By 2.
- Target score is configurable.
- Supported target scores:
  - 11
  - 15
  - 21
- Win By Two is configurable.
- Historical scoring rules are copied onto each Game when the Game is created.
- First game is generated deterministically for rotation-enabled sessions.
- Next game is generated only after the current result is successfully confirmed.
- A disputed result is not official.
- Session completion is explicit.
- Critical gameplay transitions are atomic.
- Booking = reservation.
- Playing Session = actual gameplay.

---

### 5.5 Game Result / Confirmation UX

> **Updated by U5, U17, I5/I6, I8, U15, B1 (see section 10).** Authorized Scorers can submit but cannot confirm in MVP (U5, I5/I6). The Session Host may confirm a game they played in (U17). There is no automatic timeout or confirmation (I8). Only registered players may file formal disputes; guests raise concerns through the Session Host (U15). Every resubmission creates a new Result version (B1).

**Important final decision:**

> DO NOT require every player to confirm a game result.

**Normal flow:**

1. Game finishes.
2. Session Host or authorized scorer records final score.
3. Server/domain validates score.
4. Host confirms "Game Complete".
5. Result becomes official.
6. Statistics/derived state update.
7. Rotation can generate the next game.

**Rules:**

- Final score only for MVP.
- Do not record every rally/point.
- System already knows Game Participants and Teams.
- Scorer enters only final team scores.
- Winner is derived from validated score.
- Winner is not manually selected.
- Host cannot modify the participant list through score entry.
- This is intentionally a low-friction confirmation flow.
- Players may optionally dispute a confirmed result.

---

### 5.6 Results, Confirmation & Disputes

> **Superseded in part by U1–U4, B1, U15, I1 (see section 10).** The lifecycles shown in this subsection combined Game and Result states. The authoritative lifecycles are:
>
> - **Game:** SCHEDULED → ACTIVE → COMPLETED
> - **Game Result:** SUBMITTED → CONFIRMED or SUPERSEDED; CONFIRMED → SUPERSEDED
> - **Dispute:** DISPUTED → ADMIN_REVIEW → RESOLVED
>
> DISPUTED is neither a Game status nor a Game Result status; a Dispute does not reopen the Game. Results are versioned (U1, B1). Dispute submission is limited by U15. Dispute resolution eligibility is limited by I1.

**Normal result lifecycle:**

```text
ACTIVE
→ RESULT_SUBMITTED
→ CONFIRMED
```

**Dispute path:**

```text
CONFIRMED
→ DISPUTED
→ ADMIN_REVIEW
→ RESOLVED
```

**Rules:**

- Final scores only.
- Primary scorer = Session Host.
- Authorized scorer is also supported.
- Player confirmation is optional/not required.
- Host confirms Game Complete.
- Server/domain validates the score.
- Winner is derived.
- Invalid scores are rejected.
- Confirmed result becomes official.
- Players may dispute after confirmation.
- Dispute requires a reason.
- MVP dispute reasons:
  - `INCORRECT_SCORE`
  - `WRONG_PLAYERS_OR_TEAM`
  - `OTHER`
- Description is required.
- Facility Admin resolves disputes.
- Host should not independently resolve their own disputed result.
- Disputed results are excluded from official statistics until resolved.
- Admin can uphold the original result or correct the result.
- Corrected result becomes authoritative.
- Derived statistics must be corrected/recomputed if necessary.
- Later games are NOT automatically destroyed or regenerated when an earlier result is disputed.
- Admin resolves the dispute without automatically rebuilding the later game chain.
- Preserve audit history.
- Important result/dispute actions should be auditable.
- One active dispute per Game/Result for MVP.
- Resolved disputes remain in history.

**Critical transition (should be atomic):**

```text
authorization
→ score validation
→ result confirmation
→ stats/derived state
→ rotation state
→ next game
```

---

### 5.7 Smart Rotation & Player Eligibility

> **Updated by U12, I3 (see section 10).** Session Participant status (ACTIVE / LEFT) is the sole authority for future gameplay eligibility. Check-In is attendance only.

**Smart Rotation rules:**

- Open Play Smart Rotation is automatic.
- Private Booking Smart Rotation is optional.
- Rotation is deterministic.
- No randomness.
- No LLM/AI control.
- Current active game is never changed by rotation.

**Open Play eligibility requires:**

- registered
- checked in
- available
- not already assigned to another active game

**Additional rules:**

- Late arrivals become eligible for future games.
- Players who leave are excluded from future rotation.
- Primary fairness goals:
  - fewer games played
  - longer waiting time
  - waiting duration
  - avoid consecutive games
- Variety goals:
  - avoid repeated partners when practical
  - avoid repeated opponents when practical
- Wins/losses are secondary, not primary.
- Team assignment is deterministic.
- Stable deterministic tie-breaker required.
- Multiple courts and simultaneous games are supported.
- Rotation scope is the current Playing Session.
- Global/lifetime statistics are not the primary rotation input.
- Exact numerical weights/algorithm are **(DEFERRED)** until domain algorithm implementation.
- 4 players → one doubles game.
- 5 players → 4 play + 1 wait.
- 6 players → 4 play + 2 wait.
- Minimum doubles players = 4.
- No skill rating/ELO/DUPR in MVP.

**Naming rule:**

- Do NOT call Smart Rotation "Random".
- If random team drawing is ever added, it should be a separate future feature called something such as Random Team Draw **(DEFERRED)**.

---

### 5.8 Private Booking Gameplay Modes

> **Updated by U6, U7 (see section 10).** MVP gameplay modes are SMART_ROTATION and FIXED_PARTNERS. TOURNAMENT remains a recognized future mode but is not enabled or implemented in MVP (U7). FIXED_PARTNERS initial matchup, court assignment, and subsequent matchups are defined by U6.

Private Booking gameplay has three modes:

1. `SMART_ROTATION`
2. `FIXED_PARTNERS`
3. `TOURNAMENT`

**General rules:**

- Gameplay mode belongs to Playing Session, not Booking.
- Mode is selected before gameplay begins.
- Mode should not silently change after gameplay begins.
- Open Play does not require private gameplay mode selection.
- Open Play automatically uses Smart Rotation.

**SMART_ROTATION:**

- Partners/opponents may change.
- System automatically creates deterministic assignments.
- Designed for recreational/fair rotation.

**FIXED_PARTNERS:**

- Partners stay together for the session.
- Host/players can choose partners.
- System may rotate which teams play each other.
- System may rotate court/matchups.
- Example with 8 players:

```text
A+B
C+D
E+F
G+H
```

These remain partner teams while opponents/courts can rotate.

**TOURNAMENT:**

- Fixed teams.
- Host/players may choose teams or system may generate teams.
- Competitive structure.
- Future tournament engine provides rounds/brackets/standings.

**Shared engine:**

All three modes reuse:

```text
Game
→ Score
→ Confirmation
→ Statistics
```

Tournament-specific higher-level logic remains separate from the core Game engine.

**Preferred conceptual field:** `gameplay_mode`

**Values:**

```text
SMART_ROTATION
FIXED_PARTNERS
TOURNAMENT
```

---

### 5.9 Players, Guests & Identity

> **Updated by U5, U10, U14, U15, I14 (see section 10).** One account may hold both PLAYER and FACILITY_ADMIN (U14). Guests are added directly by the Booking Owner (U10). Guests cannot be Authorized Scorers (U5) or file formal disputes (U15). Guest Identity is private-context scoped (I14).

**Authentication:**

- Supabase Auth.
- Application Profile is linked to Auth user.

**Global MVP roles:**

```text
PLAYER
FACILITY_ADMIN
```

- SESSION_HOST is contextual, not a permanent global role.
- Facility Staff is **(DEFERRED)**.

**Guests:**

- Private bookings support guest participants.
- Open Play anonymous guests are NOT MVP.
- Guests:
  - cannot book courts
  - may participate in private booking games
  - may participate in private rotation
  - do not automatically receive official persistent global statistics
  - participation history is preserved
  - future guest-to-account claim requires explicit verification
  - never automatically merge based only on matching names

**Identity keys:**

- Registered identity uses stable user ID.
- Display name is not an identity key.
- Guest identity has a stable internal guest reference.

---

### 5.10 Guest Identity

> **Updated by I14 (see section 10).** Guest Identity is associated with the relevant private Booking context and is not a global player identity.

**Conceptual fields:**

- `id`
- `display_name`
- `created_at`
- `updated_at`

**Rules:**

- Minimal data.
- No email/phone in MVP.
- Stable internal ID.
- Name is not identity key.
- Guest identity is scoped to the relevant private context.
- Do not automatically globally merge same-name guests.
- Guest participation history is preserved.
- No automatic global official statistics.
- Future account claiming requires explicit verification.
- Guest removal does not necessarily delete Guest Identity.

**Conceptual links:**

```text
Guest Identity
→ Booking Participant
→ Session Participant
→ Game Participant
```

---

### 5.11 Facility & Configuration

**Facility conceptual fields:**

- `id`
- `name`
- `description`
- `contact_email`
- `contact_phone`
- `timezone`
- `status`
- `created_at`
- `updated_at`

**Facility status:**

```text
ACTIVE
INACTIVE
```

**Rules:**

- Facility Configuration is conceptually separate.
- The product currently targets one physical facility, but the architecture keeps facility ownership/context explicit using `facility_id`.
- Do not convert this into a marketplace/multi-tenant product.
- Facility-wide closure is separate from Facility status.

---

### 5.12 Operating Hours & Facility Closures

**Operating Hours conceptual fields:**

- `id`
- `facility_id`
- `day_of_week`
- `start_time`
- `end_time`
- `created_at`
- `updated_at`

**Operating Hours rules:**

- Multiple windows per day supported.
- Closed days represented by absence of active operating-hour rows.
- Requested booking/session must fit completely within an active operating window.
- Facility timezone is used for local operational times.
- Persist timestamps in UTC.
- Overnight operating windows are not currently supported.

**Facility Closure conceptual fields:**

- `id`
- `facility_id`
- `start_time`
- `end_time`
- `reason_type`
- `description`
- `created_at`
- `updated_at`
- `cancelled_at`

**Facility Closure rules:**

- Facility closure is separate from court blockout.
- `[start,end)` interval.
- Existing future bookings conflicting with closure are flagged/reviewed.
- Do not silently change/delete those bookings.
- Cancellation retains the closure record.

---

### 5.13 Courts

**Court conceptual fields:**

- `id`
- `facility_id`
- `name`
- `status`
- `court_type`
- `description`
- `created_at`
- `updated_at`

**Court status:**

```text
ACTIVE
INACTIVE
```

**Court type:**

```text
INDOOR
OUTDOOR
```

**Rules:**

- Court name unique within facility.
- Availability is calculated.
- No availability boolean.
- Temporary closure = Court Blockout.
- Permanent/current unavailable = INACTIVE.
- Multiple photos supported.
- Historical records retained.
- Individual court hours are **(DEFERRED)**.
- Court-specific booking-rule overrides are **(DEFERRED)**.

---

### 5.14 Court Blockout

**Conceptual fields:**

- `id`
- `court_id`
- `start_time`
- `end_time`
- `reason_type`
- `description`
- `status`
- `created_by`
- `created_at`
- `updated_at`
- `cancelled_at`

**Reason types:**

```text
MAINTENANCE
PRIVATE_EVENT
OTHER
```

**Status:**

```text
ACTIVE
CANCELLED
```

**Rules:**

- Facility closure is separate.
- `[start,end)` interval.
- Admin only.
- Existing booking/Open Play conflicts are flagged/reviewed.
- Never silently rewrite existing reservations.
- Historical records retained.
- Availability calculated.
- Facility timezone for local operation, UTC persistence.
- Prefer cancellation over physical deletion.

---

### 5.15 Court Photos

**Conceptual fields:**

- `id`
- `court_id`
- `storage_path`
- `alt_text`
- `display_order`
- `is_primary`
- `created_at`
- `updated_at`

**Rules:**

- Actual image file stored in Supabase Storage.
- Database stores metadata.
- Multiple photos per court.
- At most one primary photo.
- Display order supported.
- Alt text optional.
- Photo metadata/file may be physically deleted.
- Storage object should be cleaned up when record is deleted.
- If primary is deleted and others remain, next ordered photo should preferably become primary.
- Admin manages.
- Players/users read.
- Exact storage bucket/path **(DEFERRED)**.

---

### 5.16 Booking

> **Updated by U11, I12, I7, U18 (see section 10).** Payment processing and payment-dependent booking state are excluded from MVP (U11, I12). The Booking Owner is automatically an active Booking Participant (I7). Booking end is a hard boundary for the private Playing Session (U18).

**Conceptual fields:**

- `id`
- `facility_id`
- `court_id`
- `owner_id`
- `start_time`
- `end_time`
- `status`
- `created_at`
- `updated_at`
- `cancelled_at`
- `completed_at`

**Status:**

```text
CONFIRMED
CANCELLED
COMPLETED
```

**Rules:**

- Owner is required registered user.
- Guest cannot own booking.
- `[start,end)`.
- Facility timezone applies to operational interpretation.
- Server validation checks:
  - authentication
  - authorization/role
  - facility active
  - court belongs to facility
  - court active
  - valid interval
  - duration policy
  - advance booking window
  - operating hours
  - facility closure
  - court blockout
  - court booking conflict
  - maximum future booking rule
- Cancellation retains record.
- Cancellation deadline configurable.
- Completion occurs after scheduled period.
- Booking does not automatically start gameplay.
- At most one Playing Session per Booking.
- Database + server prevent concurrency/double booking.
- Notification failure must not roll back booking.

**Booking configuration terms:**

- minimum booking duration
- maximum booking duration
- advance booking window
- cancellation deadline
- maximum future bookings

These are Facility Configuration values, not hardcoded product constants.

---

### 5.17 Booking Participants

> **Updated by U10, I7, I14 (see section 10).** Registered players (other than the owner) join only through an accepted Booking Invitation (U10). The Booking Owner is automatically created as an active REGISTERED Booking Participant with no invitation (I7); `booking.owner_id` remains the ownership authority. Guests are added directly by the Booking Owner and are private-context scoped (U10, I14).

**Conceptual fields:**

- `id`
- `booking_id`
- `user_id`
- `guest_id`
- `participant_type`
- `status`
- `joined_at`
- `removed_at`
- `created_at`
- `updated_at`

**Participant type:**

```text
REGISTERED
GUEST
```

**Status:**

```text
ACTIVE
REMOVED
```

**Rules:**

- Registered participant uses `user_id`.
- Guest participant uses `guest_id`.
- User/guest identity is mutually exclusive.
- Stable guest reference required.
- Booking owner remains `booking.owner_id`.
- Do not duplicate owner flag here.
- Session Host is session-level, not participant-level.
- Physical deletion should generally be avoided.
- Once participant has played a confirmed game, historical participation cannot be deleted.
- Late participants may join future games only.
- Early leavers are excluded from future rotation.
- Booking Participant ≠ Session Participant ≠ Game Participant.

---

### 5.18 Playing Session

> **Updated by U5, I5/I6, U6, U7, U18, U19, I4(c) (see section 10).** Authorized Scorers are designated at Playing Session level (U5, I5/I6). TOURNAMENT origin/mode is future only (U7). FIXED_PARTNERS first-game generation follows U6. Booking end is a hard boundary (U18). The Open Play host is a FACILITY_ADMIN designated as Open Play Host (U19). No Playing Session is created for an Open Play with fewer than 4 eligible checked-in players (I4(c)).

**Conceptual fields:**

- `id`
- `facility_id`
- `origin_type`
- `booking_id`
- `open_play_id`
- `host_id`
- `gameplay_mode`
- `status`
- `started_at`
- `ended_at`
- `created_at`
- `updated_at`

**Origin:**

```text
BOOKING
OPEN_PLAY
TOURNAMENT
```

**Status:**

```text
CREATED
READY
ACTIVE
COMPLETED
```

**Rules:**

- Exactly one valid origin.
- Private session host = Session Host.
- Open Play host = facility admin/staff/designated Open Play host.
- Minimum four eligible doubles players.
- Explicit session start.
- Private gameplay mode required.
- Open Play automatically uses Smart Rotation.
- Mode should not silently change mid-session.
- Multiple courts supported.
- Session does not have one single `court_id`.
- Game owns actual gameplay court.
- Maximum one active Game per court/session.
- First game generated deterministically when rotation-enabled session starts.
- Next game only after previous result is confirmed.
- Explicit session completion.
- No unresolved active game when completing.
- Private session cannot extend beyond booking end.
- At most one Playing Session per Booking.
- Historical host/mode/game information retained.

---

### 5.19 Session Participants

> **Updated by U12, I3 (see section 10).** `Session Participant.status` (ACTIVE / LEFT) is the sole authority for future gameplay eligibility. Checking out (Open Play) causes the Session Participant to become LEFT.

**Conceptual fields:**

- `id`
- `session_id`
- `booking_participant_id`
- `open_play_registration_id`
- `participant_type`
- `user_id`
- `guest_id`
- `status`
- `joined_at`
- `left_at`
- `created_at`
- `updated_at`

**Status:**

```text
ACTIVE
LEFT
```

**Rules:**

- Private source = Booking Participant.
- Open Play source = Open Play Registration after check-in.
- Source must match session origin.
- Open Play player must be checked in before normal session participation.
- Current playing assignment is represented by Game Participant.
- Late arrivals can participate in future games.
- Early leavers become ineligible for future games.
- Player cannot be in two active games simultaneously in same session.
- Eligibility is dynamically evaluated.
- Session Host stored on Playing Session.
- Lifetime stats are not stored here.
- Session Participants feed Smart Rotation.

---

### 5.20 Game

> **Superseded in part by U2, U3, U4, I8 (see section 10).** Game status values are SCHEDULED, ACTIVE, COMPLETED only. RESULT_SUBMITTED, CONFIRMED, and DISPUTED are no longer Game statuses: result state belongs to Game Result and dispute state belongs to Dispute. A Game remains COMPLETED after a dispute, upheld result, or correction. A SUBMITTED Result is never automatically confirmed (I8).

**Conceptual fields:**

- `id`
- `session_id`
- `court_id`
- `game_number`
- `status`
- `target_score`
- `win_by_two`
- `started_at`
- `ended_at`
- `created_at`
- `updated_at`

**Game status:**

```text
SCHEDULED
ACTIVE
RESULT_SUBMITTED
CONFIRMED
DISPUTED
```

**Rules:**

- Game number unique within session.
- Court required.
- Multiple simultaneous games supported across different courts.
- At most one ACTIVE game per court within session.
- Player cannot be in two simultaneous active games.
- Scoring rules copied from Facility Configuration at Game creation.
- Target score supports 11/15/21.
- Win By Two configurable.
- Winner derived from validated Result.
- No rally history MVP.
- Final score only.
- Host/authorized scorer submits result.
- Host confirms Game Complete.
- Player dispute is optional.
- Disputed game is not official for stats until resolved.
- Next game only after confirmed result.
- First game generated deterministically.
- Result → Stats → Rotation → Next Game transition should be atomic.

---

### 5.21 Game Teams

**Conceptual fields:**

- `id`
- `game_id`
- `team_number`
- `created_at`

**Rules:**

- Normally two teams: 1 and 2.
- Team identity is game-specific.
- Players are linked through Game Participants.
- Do not put player IDs directly on Team.
- Do not store score on Team.
- Do not store winner flag on Team.
- Smart Rotation teams may change each game.
- Fixed Partner teams persist at session level.
- Tournament teams are future higher-level structures.
- Team assignment should not casually change after gameplay begins.
- Flexible enough for future singles.
- No team names required.
- `team_number` is deterministic.

---

### 5.22 Game Participants

> **Updated by U16 (see section 10).** Wrong player/team assignments are corrected non-destructively by a FACILITY_ADMIN; the original assignment is preserved and the corrected assignment becomes authoritative. The physical representation is a schema-design decision.

**Conceptual fields:**

- `id`
- `game_id`
- `team_id`
- `session_participant_id`
- `participant_type`
- `user_id`
- `guest_id`
- `created_at`

**Rules:**

- Exactly one team per Game Participant.
- Participant appears at most once per Game.
- Participant cannot belong to both teams.
- Registered participant uses `user_id`.
- Guest uses `guest_id`.
- Must link to Session Participant from same session.
- Waiting players have no Game Participant record for that Game.
- Player cannot be assigned to simultaneous active games in same session.
- Assignments are effectively immutable once gameplay begins.
- Confirmed participation feeds official statistics for registered users.
- Guest participation is preserved without automatic global stats.
- Smart Rotation uses Game Participant history.
- No score/winner/team name stored here.

---

### 5.23 Game Results

> **Superseded in part by U1, U2, U3, B1 (see section 10).**
>
> - A Game may have multiple Game Result versions. No Result is ever overwritten.
> - Every resubmission or correction creates a new version, and the previous version becomes SUPERSEDED — including a version that is still SUBMITTED (B1).
> - Only one Result version can be authoritative at a time: the current CONFIRMED version.
> - The `dispute_id` field is removed; Dispute references the Result (U1).
> - "One current Result per Game" is read as "one authoritative Result per Game".

Game Result is separate from Game.

**Conceptual fields:**

- `id`
- `game_id`
- `team_1_score`
- `team_2_score`
- `submitted_by`
- `submitted_at`
- `confirmed_by`
- `confirmed_at`
- `status`
- `dispute_id` (optional conceptual relationship)
- `created_at`
- `updated_at`

**Rules:**

- Final team scores only.
- Primary scorer = Session Host.
- Authorized scorer supported.
- Host confirms Game Complete.
- Player confirmation not required.
- Score validation performed server/domain side.
- Winner derived from score.
- Historical scoring rules are stored on Game.
- Target score 11/15/21.
- Win By Two configurable.
- One current Result per Game.
- Normal lifecycle `RESULT_SUBMITTED → CONFIRMED`.
- Dispute path from confirmed result.
- No rally history.
- Result does not define player/team membership.

---

### 5.24 Disputes

> **Updated by U1, U2, U3, U15, I1 (see section 10).** A Result may have 0..N Disputes over time with at most one active Dispute (U1). Formal disputes are submitted by registered participants; the Session Host may submit on a guest's behalf (U15). A FACILITY_ADMIN who is the Session Host, the Open Play Host, or a participant in the disputed Game cannot resolve that Dispute (I1).

**Conceptual fields:**

- `id`
- `game_id`
- `result_id`
- `reason`
- `description`
- `status`
- `submitted_by`
- `resolved_by`
- `created_at`
- `updated_at`
- `resolved_at`

**Dispute reasons:**

```text
INCORRECT_SCORE
WRONG_PLAYERS_OR_TEAM
OTHER
```

**Lifecycle:**

```text
DISPUTED
→ ADMIN_REVIEW
→ RESOLVED
```

**Rules:**

- Separate entity from Game Result.
- Only game participants can submit normal disputes.
- Reason required.
- Description required.
- Facility Admin resolves.
- Host cannot independently resolve own disputed result.
- Disputed result excluded from official statistics.
- Original Game, Teams, Participants, Result and timestamps preserved.
- Admin can uphold or correct result.
- Corrected result becomes authoritative.
- Derived statistics corrected/recomputed.
- Later games are not automatically deleted/regenerated.
- One active dispute per Game/Result for MVP.
- Resolved disputes retained.
- Important dispute actions audited.
- Players cannot directly edit historical Game Participants or official Results.

---

### 5.25 Open Play

> **Updated by U8, B2, U9, U13, U19, I4(c), I12 (see section 10).** Adds an informational skill level (U9) and a reference to the originating Recurring Open Play Schedule for generated instances (U8, B2). Adds terminal CANCELLED state (U13). `host_id` must reference an authorized FACILITY_ADMIN (U19). The fee is informational only (I12). Fewer than 4 eligible checked-in players at SESSION_READY: no Playing Session, may transition to COMPLETED (I4(c)).

**Conceptual fields:**

- `id`
- `facility_id`
- `title`
- `description`
- `start_time`
- `end_time`
- `registration_open_at`
- `registration_close_at`
- `capacity`
- `fee`
- `status`
- `host_id`
- `created_by`
- `created_at`
- `updated_at`
- `cancelled_at`

**Status:**

```text
SCHEDULED
REGISTRATION_OPEN
SESSION_READY
ACTIVE
COMPLETED
```

**Rules:**

- Facility-created.
- Individual registration.
- No partner required.
- Capacity is registration capacity.
- No waitlist MVP.
- Registration ≠ check-in ≠ playing.
- Normal check-in requires registration.
- Checked-in players become eligible.
- Self check-in supported.
- Admin/staff can correct.
- Host explicitly starts.
- Minimum four eligible checked-in players for doubles.
- No automatic clock-based start.
- Zero checked-in players can end without games.
- Explicit end.
- No anonymous Open Play guests MVP.
- Multiple courts supported.
- Smart Rotation automatic.
- Recurring Open Play creates separate instances.
- Cancelled Open Play cannot start gameplay.
- History retained.

---

### 5.26 Open Play Courts

> **Updated by B2 (see section 10).** Court reservations of generated recurring instances are validated against operating hours, court availability, bookings, blockouts, and facility closures; conflicts require FACILITY_ADMIN review and are never silently resolved.

**Conceptual fields:**

- `id`
- `open_play_id`
- `court_id`
- `created_at`
- `updated_at`

**Rules:**

- Many-to-many relationship between Open Play and Court.
- Open Play can reserve one or more courts.
- Unique court per Open Play.
- Court must belong to same facility.
- Court must be active when reservation is validated.
- Entire Open Play interval must fit court availability.
- Conflict checks include:
  - bookings
  - other Open Plays
  - court blockouts
  - facility closures
  - court status
  - operating hours
- `[start,end)`.
- This represents reserved court capacity, not actual gameplay.
- Game owns actual gameplay court.
- Playing Session does not have a single court.
- Reserved courts may be changed before gameplay subject to validation.
- Active gameplay/history must not be casually altered.
- Cancelled Open Play retains reservation history.
- Smart Rotation can only assign games to reserved courts.

---

### 5.27 Open Play Registration

**Conceptual fields:**

- `id`
- `open_play_id`
- `user_id`
- `status`
- `registered_at`
- `cancelled_at`
- `created_at`
- `updated_at`

**Status:**

```text
REGISTERED
CANCELLED
```

**Rules:**

- Authenticated users only.
- No anonymous guests.
- Stable `user_id` identity.
- One active registration per player/Open Play.
- Immediate confirmation after validation.
- No pending approval.
- No waitlist.
- Registration window enforced.
- Capacity enforced atomically.
- Registration ≠ check-in ≠ gameplay.
- Normal check-in requires registration.
- Cancellation supported and retained.
- Cancelled player may re-register if rules/capacity permit.
- Registration does not create official stats.
- Notification failure does not roll back registration.
- Player cannot bypass capacity/rules.
- Normal registration closes according to deadline.
- No normal registration after gameplay begins.

---

### 5.28 Open Play Check-In / Attendance

> **Updated by U12, I3, U14 (see section 10).** Check-In status values are CHECKED_IN and CHECKED_OUT. Check-In represents attendance only. For future gameplay eligibility, `Session Participant.status` is the sole authority; the eligibility list in this subsection is superseded for that purpose (I3). Checking out causes the Session Participant to become LEFT and does not terminate an active Game. "Admin/staff" means FACILITY_ADMIN in MVP.

**Conceptual fields:**

- `id`
- `open_play_registration_id`
- `checked_in_at`
- `checked_in_by`
- `status`
- `checked_out_at`
- `created_at`
- `updated_at`

**Rules:**

- Normal check-in requires valid registration.
- No anonymous Open Play guests.
- Self check-in allowed.
- Admin/staff can record/correct.
- No duplicate active check-ins.
- Check-in provides basis for eligibility.
- Eligibility dynamically considers:
  - registered
  - checked in
  - available
  - not checked out
  - not currently playing
- Late arrivals future games only.
- Early departure excludes future rotation.
- Check-out does not modify/terminate active game.
- Attendance contributes to Open Play sessions attended.
- Confirmed Game Participant contributes to Games Played.
- Attendance retained.
- Exact separate check-in window is **(DEFERRED)**.
- Notification failure does not affect attendance.
- Admin corrections preserve history.
- Check-in ≠ Session Participant ≠ Game Participant.

---

### 5.29 Player Statistics

> **Updated by U1, U4, U16, I14 (see section 10).** Official statistics use the authoritative CONFIRMED Game Result (not under an active Dispute) together with the authoritative Game participation (including U16 corrections). Guests do not receive persistent global statistics (I14).

**Authoritative source:**

```text
Confirmed Game
+
Game Participants
+
Game Result
```

**Rules:**

- Only confirmed results count.
- Disputed results excluded until resolved.
- Games Played = confirmed games participated in.
- Wins/Losses derived from confirmed result.
- Win Rate = Wins / Games Played.
- Zero games displays —.
- Points Scored = player's team final scores.
- Points Against = opponent final scores.
- Point Differential = Points Scored − Points Against.
- Open Play Sessions Attended = attendance/check-in based.
- Court Sessions Played = actual Playing Session participation.
- Partners and opponents derived from confirmed games.
- Recent Results derived from confirmed games.
- Lifetime and session views are supported.
- Private Booking and Open Play use the same statistics architecture.
- Tournament-specific metrics are **(DEFERRED)**.
- Guests do not automatically receive persistent global official stats.
- Cached stats may be used for performance.
- Cached stats are never authoritative.
- Cached stats must be rebuildable from confirmed games/results.
- Statistics and Smart Rotation are separate systems.
- Wins/losses are not the primary rotation input.
- Avoid redundant authoritative stat storage when values are cheap to derive.
- Corrected disputes must correct derived/cached statistics.

---

### 5.30 Notifications

> **Updated by U10 (see section 10).** The notification system can notify invited players. The type list in this subsection remains a list of potential types; exact type names are finalized during schema design.

Notifications communicate system state but do not define system state.

**Conceptual fields:**

- `id`
- `user_id`
- `type`
- `title`
- `message`
- `read_at`
- `created_at`
- `updated_at`

**Optional structured reference:**

- `related_entity_type`
- `related_entity_id`

**MVP channels:**

- In-app
- Email

**Potential notification types:**

```text
BOOKING_CONFIRMED
BOOKING_CANCELLED
OPEN_PLAY_REGISTRATION_CONFIRMED
OPEN_PLAY_REGISTRATION_CANCELLED
OPEN_PLAY_CANCELLED
OPEN_PLAY_SESSION_READY
SESSION_STARTED
GAME_RESULT_CONFIRMED
GAME_RESULT_DISPUTED
DISPUTE_SUBMITTED
DISPUTE_RESOLVED
ANNOUNCEMENT
```

**Rules:**

- Recipient is authenticated user.
- `read_at` null = unread.
- User can mark own notification as read.
- Users cannot modify notification ownership/content.
- Notifications retained.
- No automatic expiration MVP.
- Notification preferences **(DEFERRED)**.
- Email failure never rolls back core transaction.
- Core state commits before success notification.
- Notifications may reference related domain entities.
- Access restricted to authorized recipient.
- Notifications are separate from Facility Announcements.
- Notifications never control booking/scoring/rotation/stats/core writes.

---

### 5.31 Facility Announcements

**Conceptual fields:**

- `id`
- `facility_id`
- `title`
- `content`
- `status`
- `published_at`
- `created_by`
- `created_at`
- `updated_at`
- `archived_at`

**Status:**

```text
DRAFT
PUBLISHED
ARCHIVED
```

**Rules:**

- Facility Admin creates/manages.
- Players cannot create announcements.
- Published announcements are visible according to application UI.
- Announcements may generate notifications.
- Notification delivery failure does not affect announcement state.
- Announcements do not control booking availability.
- Facility Closures remain authoritative for closures.
- Court Blockouts remain authoritative for court closures.
- Historical announcements retained.
- Archive instead of normal physical deletion.
- Scheduled publishing **(DEFERRED)**.
- Rich-text/editor details **(DEFERRED)**.
- Announcement notification preferences **(DEFERRED)**.

---

### 5.32 Audit History

> **Updated by U1, U4, U5, U16 (see section 10).** Score submission, game confirmation, administrative intervention, dispute resolution, result corrections, and participant/team corrections should be recorded in audit history.

**Conceptual fields:**

- `id`
- `facility_id`
- `actor_user_id`
- `action_type`
- `entity_type`
- `entity_id`
- `description`
- `metadata`
- `created_at`

**Rules:**

- Records meaningful state-changing system actions.
- Separate from Notifications.
- Not the domain state source of truth.
- Append-oriented.
- Historical audit records retained.
- Users cannot edit/delete audit records.
- Facility Admin can view authorized audit history.
- Player/host visibility depends on future authorization/UI **(DEFERRED)**.
- Important actions should be audited:
  - booking actions
  - court actions
  - facility actions
  - Open Play actions
  - gameplay actions
  - disputes
  - result corrections
  - announcements
- Do not duplicate every derived statistic.
- System actions may have no human actor.
- Metadata can contain useful before/after context.
- Never store secrets or unnecessary sensitive information in audit metadata.
- Exact action/entity types finalized during schema design **(DEFERRED)**.

---

### 5.33 Authentication & Profiles

> **Updated by U14 (see section 10).** One account may hold both PLAYER and FACILITY_ADMIN. SESSION_HOST, Open Play Host, and Authorized Scorer are contextual, not global roles. Exact physical role/capability representation is a schema-design decision.

**Rules:**

- Supabase Auth is authentication authority.
- PickleHub does not store passwords or auth session credentials.
- Each authenticated user has an application Profile.
- Stable identity = `user_id`.
- Display name is not identity key.

**Global roles:**

```text
PLAYER
FACILITY_ADMIN
```

**Profile status:**

```text
ACTIVE
DEACTIVATED
```

**Additional rules:**

- SESSION_HOST is contextual, not global.
- Facility Staff **(DEFERRED)**.
- Normal registration creates PLAYER.
- Users cannot self-assign FACILITY_ADMIN.
- Deactivation preserves historical data.
- Guest Identity separate from Profile.
- Guest-to-account conversion requires explicit future verification.
- Statistics reference user identity.
- Notifications reference `user_id`.
- Audit records reference `actor_user_id`.
- Authentication and authorization are separate.
- Server-side authorization required.
- RLS may provide database-level protection where appropriate.
- Exact auth providers beyond MVP **(DEFERRED)**.

---

### 5.34 Facility Configuration

Facility Configuration is separate from Facility identity. It stores operational/business defaults.

**It does NOT duplicate:**

- facility name
- facility timezone
- operating hours
- court records
- court-specific hours

**Booking configuration:**

- minimum booking duration
- maximum booking duration
- advance booking window
- cancellation deadline
- maximum future bookings

**Gameplay defaults:**

- default target score
- default Win By Two

**Supported target scores:**

```text
11
15
21
```

Win By Two is configurable.

**Rules:**

- Configuration provides defaults/rules for new operations.
- Historical Games preserve their own scoring rules.
- Configuration changes do not rewrite historical games.
- Facility Admin manages configuration.
- Players may read relevant configuration but cannot modify it.
- Configuration changes should be audited.
- Use explicit typed fields rather than generic key/value for MVP.
- Physical database types/units are schema-design decisions **(DEFERRED)**.

---

### 5.35 Session Teams / Fixed Partners

> **Updated by U6 (see section 10).** The host configures teams before gameplay; PickleHub deterministically generates the initial team-vs-team matchup and court assignment, which the host may review and adjust before the session starts; subsequent matchups are generated deterministically.

Fixed Partners requires persistent session-level teams.

**Session Team:**

- Belongs to one Playing Session.
- Exists only for Fixed Partners mode.
- Session Team Member links team to Session Participant.
- Team does not directly reference user/guest.
- MVP fixed partner teams contain two participants.
- Registered users and guests may belong.
- Teams are configured before gameplay.
- Partners can change before gameplay.
- Once gameplay begins, fixed-partner configuration is treated as locked.
- Leaving ends active team membership but preserves history.
- Historical Game Participants are authoritative for actual gameplay.
- Departed partner is not automatically replaced.
- Smart Rotation does not use Session Teams.
- Tournament has separate future team structure.

**Session Team Member:**

- Links Session Team → Session Participant.
- Participant can belong to at most one active fixed team in a session.
- MVP team size = two active participants.
- Team assignments can be created/changed before gameplay.
- DB/server validation prevents inconsistent relationships.
- Does not directly reference user/guest.

---

### 5.36 Booking → Playing Session Relationship

> **Updated by I7, U18 (see section 10).** The Booking Owner is automatically an active Booking Participant (I7). Booking end is a hard reservation boundary; no new Game starts after it, and an active Game reaching it remains unresolved and prevents normal Session completion (U18).

- Booking = court reservation.
- Playing Session = actual gameplay.
- One Booking can have at most one Playing Session.
- Booking does not automatically create a session.
- Session creation/start is explicit.
- Booking Owner defaults to Session Host.
- Host can be transferred before gameplay subject to authorization.
- Booking Participants and Session Participants are separate.
- Not every Booking Participant must become a Session Participant.
- Session Participants represent actual gameplay participation.
- Minimum four eligible doubles players.
- Booking and Session timestamps are separate.
- Session may start late.
- Session cannot extend beyond Booking end.
- Booking cancellation before gameplay prevents session start.
- Historical gameplay remains preserved once it occurred.
- Booking completion and Session completion are separate.
- Gameplay mode belongs to Session.
- Booking owns reserved court.
- Game owns actual gameplay court.
- Historical participation/game records must not be casually deleted/rewritten.

---

### 5.37 Open Play → Playing Session Relationship

> **Updated by U19, I4(c), U13, I3 (see section 10).** "Host" means the designated Open Play Host, who must be an authorized FACILITY_ADMIN (U19). With fewer than 4 eligible checked-in players at SESSION_READY, no Playing Session is created and the Open Play may transition to COMPLETED (I4(c)). A CANCELLED Open Play cannot create a Playing Session (U13).

- Open Play = scheduled facility activity.
- Playing Session = actual gameplay.
- One Open Play can have at most one Playing Session.
- Open Play does not automatically create/start gameplay.
- Host explicitly starts.
- Minimum four eligible checked-in players.
- Registration ≠ Check-In ≠ Session Participant ≠ Game Participant.
- Open Play automatically uses Smart Rotation.
- Multiple courts supported.
- Open Play Courts represent reserved courts.
- `Game.court_id` represents actual gameplay court.
- Late arrivals can join future games.
- Early departures excluded from future rotation.
- Zero checked-in players can end without games.
- Session and Open Play completion are separate explicit transitions.
- Cancelled Open Play cannot start gameplay.
- Confirmed games drive official stats.
- Check-in drives attendance.
- Notification/audit failures do not change gameplay state.

---

## 4. Relationship & Integrity Review

**Status: LOCKED** (conceptual relationships; physical keys and constraints are DEFERRED)

### 4.1 Facility

```text
Facility
→ Configuration (1:1)
→ Operating Hours (1:N)
→ Facility Closures (1:N)
→ Courts (1:N)
→ Announcements (1:N)
```

### 4.2 Court

```text
Court
→ Photos (1:N)
→ Blockouts (1:N)
→ Bookings (1:N)
```

### 4.3 Identity

```text
Supabase Auth User
→ Profile (1:1)
```

Guest Identity remains separate.

### 4.4 Booking

```text
Profile
→ Bookings

Court
→ Bookings

Booking
→ Booking Participants (1:N)
→ Playing Session (0..1)
```

### 4.5 Open Play

> **Updated by U8, B2 (see section 10).** Added: `Recurring Open Play Schedule → Open Play instances (0..N)`. Each generated instance retains a reference to its originating schedule and is otherwise independent.

```text
Open Play
→ Registrations (1:N)
→ Open Play Courts (1:N)
→ Playing Session (0..1)

Registration
→ Check-In (0..1)
```

### 4.6 Playing Session

```text
Playing Session
→ Session Participants (1:N)
→ Session Teams (0..N, Fixed Partners only)
→ Games (1:N)

Session Team
→ Session Team Members
→ Session Participants
```

### 4.7 Game

> **Superseded in part by U1, B1 (see section 10).** `Game → Game Result` is 0..N historical Result versions with at most one authoritative version (not 0..1). `Result → Dispute` is 0..N over time with at most one active Dispute (not 0..1). Game Result does not hold a `dispute_id`.

```text
Game
→ Game Teams (1:N)
→ Game Participants (1:N)
→ Game Result (0..1)

Game Participant
→ Session Participant

Result
→ Dispute (0..1)
```

### 4.8 Downstream and Cross-Cutting Systems

- Statistics are downstream from confirmed Games/Participants/Results.
- Notifications and Audit are cross-cutting systems, not domain state authorities.

---

## 5. Major Integrity Rules

**Status: LOCKED** (rules; physical enforcement mechanisms are DEFERRED)

> **Updated (see section 10).** "One current Result per Game" means one authoritative Result per Game, with superseded versions retained (U1, B1). "Only eligible participants may dispute" is defined by U15. Dispute resolution is additionally limited by the I1 conflict-of-interest rule.

- Facility-owned entities must belong to the correct facility.
- Court name unique within facility.
- No overlapping confirmed bookings for the same court.
- Open Play capacity enforced atomically.
- One active registration per player/Open Play.
- Valid check-in required.
- Exactly one valid Playing Session origin.
- Session Participant source must match session origin.
- Session Team/Member must belong to same session.
- One active Game per court per session.
- Player cannot be in two active games.
- One fixed-team membership per participant.
- Valid score required.
- One current Result per Game.
- Only eligible participants may dispute.
- Only confirmed results count toward official statistics.
- Historical records preserved.
- Critical transitions atomic.
- Next-game generation protected/idempotent.
- Explicit state transitions.
- Audit history retained.
- Dispute history retained.

---

## 6. Redundancy / Source of Truth Rules

**Status: LOCKED**

Do not duplicate values when they can safely be derived.

Examples:

- Winner is derived from Result.
- Win rate is derived/cached, not authoritative.
- Availability is calculated.
- Identity is based on stable IDs.
- Scores live in Game Result.
- Session host lives on Playing Session.
- Booking owner lives on Booking.
- Confirmed Games/Results/Participants are authoritative for statistics.
- Cached statistics are rebuildable.
- Notifications do not define domain state.
- Audit history does not define domain state.

---

## 7. Historical Data Rules

**Status: LOCKED**

Historical records must be preserved.

Do not casually delete or rewrite:

- bookings that already occurred
- cancelled bookings
- Open Play history
- session history
- games
- game participants
- confirmed results
- disputes
- resolved disputes
- attendance
- announcements
- audit history

Historical Game scoring rules remain fixed even if Facility Configuration later changes.

Historical gameplay relationships remain authoritative.

---

## 8. Deferred Decisions

**Status: DEFERRED**

The following remain intentionally deferred unless explicitly resolved elsewhere:

- Exact Smart Rotation numerical weights.
- Exact Smart Rotation implementation algorithm.
- Skill rating/ELO/DUPR.
- Tournament-specific database structure.
- Tournament rounds/brackets/standings.
- Court-specific operating hours.
- Court-specific booking-rule overrides.
- Exact Court Photo storage bucket/path.
- Exact check-in time window.
- Facility Staff role.
- Additional authentication providers.
- Notification preferences.
- Rich-text/editor format for announcements.
- Scheduled announcement publishing.
- Delivery tracking expansion for notifications.
- Detailed audit action-type catalog.
- Physical database types/units.
- Exact indexes/constraints/triggers/functions.
- Exact Supabase RLS policies.
- Exact transaction implementation.
- Cached-stat implementation details.
- Advanced Fixed Partner incomplete-team handling.
- Advanced matchup scheduling.
- Random Team Draw future feature.

Do NOT turn deferred decisions into invented requirements.

### 8.1 Additional Deferred Items from U1–U19 and Final Consistency Decisions

- Exact Fixed Partners matchup optimization algorithm (U6).
- Tournament mode and Tournament Engine (U7).
- Recurrence generation horizon, advanced recurrence syntax, and complex recurrence patterns (U8, B2).
- Invitation expiration rules (U10).
- Payment domain: online payment, refunds, payment tracking (U11, I12).
- Rejoining after leaving; exact handling of unexpected mid-game departure; whether checkout is mandatory at session completion (U12).
- Exact physical role/capability representation (U14).
- Exact physical representation of participant/team corrections (U16).
- Exact administrative handling of an overrun Game at Booking end (U18).
- Result timeout, reminder, and escalation behavior (I8).
- Future guest-to-account claiming with explicit identity verification (I14).

---

## 9. Important Terminology

Use these terms consistently:

- Smart Rotation
- Fixed Partners
- Tournament
- Playing Session
- Session Participant
- Booking Participant
- Game Participant
- Game Team
- Game Result
- Open Play Registration
- Open Play Check-In
- Court Blockout
- Facility Closure
- Facility Configuration
- Session Host
- Facility Admin
- Guest Identity

Do not replace "Smart Rotation" with "Random".

Additional terms introduced by the resolved decisions (section 10):

- Authorized Scorer
- Open Play Host
- Recurring Open Play Schedule
- Booking Invitation
- Result version (authoritative / SUPERSEDED)

---

## 10. Resolved Decisions U1–U19 + Final Consistency Decisions

**Status: LOCKED — APPROVED**

These decisions resolve the unresolved items identified by the Step 5 consistency audits. They take precedence over section 3 where they differ.

### U1 — Corrected Result Storage

**Decision:** PickleHub preserves the original Game Result and creates a new corrected Result version when an official result needs to be corrected. A Game may have multiple historical Game Result versions, but only one Result version can be authoritative at a time.

**Rules:**

- The original Result is never overwritten.
- A correction creates a new Result version.
- The previous authoritative Result becomes non-authoritative.
- Only the current authoritative Result contributes to official statistics.
- A disputed Result is not eligible for official statistics while the dispute is unresolved.
- When an admin corrects a result, the new Result version becomes authoritative.
- Previous Result versions remain preserved for history/audit purposes.
- Existing Dispute records remain preserved.
- Later Games are not automatically deleted or regenerated because a previous result is corrected.
- Audit history records the correction/resolution.

**Relationship change:** Game Result → Dispute is 0..N disputes over the lifetime of Results, with at most one active dispute at a time. The redundant `dispute_id` field is not maintained on Game Result; the Dispute references the Result.

**Core principle:** Never overwrite historical result data; create a corrected version and change which version is authoritative.

### U2 — Game Result Lifecycle

**Decision:** Game, Game Result, and Dispute have separate lifecycles.

```text
Game:         SCHEDULED → ACTIVE → COMPLETED
Game Result:  SUBMITTED → CONFIRMED → SUPERSEDED   (see B1 for SUBMITTED → SUPERSEDED)
Dispute:      DISPUTED → ADMIN_REVIEW → RESOLVED
```

**Rules:**

- DISPUTED is not a Game status.
- DISPUTED is not a Game Result status.
- Dispute is its own entity and lifecycle.
- A corrected Result creates a new Result version.
- The previous Result becomes SUPERSEDED.
- Only the current authoritative confirmed Result is used for official statistics.
- Historical Result versions remain preserved.

**Core principle:** Game state, Result state, and Dispute state are separate concepts and must not be combined into one lifecycle.

### U3 — Game vs Game Result Authority

**Decision:**

- Game is authoritative for gameplay state.
- Game Result is authoritative for the recorded score/result.
- Dispute is authoritative for the dispute process.

The application/service layer coordinates these states:

```text
Game ACTIVE
→ Score submitted
→ Result SUBMITTED
→ Host confirms
→ Result CONFIRMED
→ Game COMPLETED
→ Stats updated
→ Next game generated
```

**Rule:** A dispute does not reopen the Game. If a completed Game's Result is disputed: Game = COMPLETED, Result = CONFIRMED, Dispute = DISPUTED. The dispute determines whether the Result remains eligible as the official statistical result.

**Core principle:** Gameplay history and result authority are separate.

### U4 — Game State After Dispute Resolution

**Decision:** A Game remains COMPLETED after gameplay ends, regardless of whether its Result is later disputed, upheld, or corrected. A dispute does not reopen or reset the Game.

- **Upheld without correction:** the existing authoritative Result remains authoritative (Game = COMPLETED, Result = CONFIRMED, Dispute = RESOLVED).
- **Corrected:** a new Result version is created as CONFIRMED and authoritative; the original Result becomes SUPERSEDED; the Game remains COMPLETED.

**Statistics:** If a correction changes the official score, the new authoritative Result becomes the source for official statistics and statistics are recalculated/updated. The old Result, the Dispute, and audit history remain in history.

**Later Games** are not automatically deleted, regenerated, rescheduled, or rolled back because an earlier Result was corrected.

**Core principle:** Gameplay history is immutable; result authority can change.

### U5 — Authorized Scorer & Result Confirmation

**Decision:** The Session Host is the primary scorer and confirmer of Game Results. A Playing Session may also designate one or more Authorized Scorers.

| Action | Session Host | Authorized Scorer | Facility Admin |
|---|---|---|---|
| Submit final score | Yes | Yes | Yes |
| Confirm Game Complete | Yes | No (see I5/I6) | Yes |
| Resolve dispute | No | No | Yes (subject to I1) |

**Authorized Scorer:** designated at the Playing Session level; must be a registered user; can submit the final score; cannot confirm Game completion; does not receive a global system-wide role; is not a substitute for the Session Host's confirmation authority.

**Facility Admin:** may submit a score when necessary, confirm a Game when necessary, perform administrative intervention, and resolve disputes.

**Guests** cannot be Authorized Scorers in MVP.

**Dispute authority:** Dispute resolution remains an administrative function. The Session Host and Authorized Scorer cannot resolve a dispute involving their own result.

**Audit:** score submission, game confirmation, administrative intervention, and dispute resolution should be recorded in audit history.

**Core principle:** Scoring submission and Game completion confirmation are separate actions, with the Session Host holding normal confirmation authority.

### U6 — Fixed Partners: First Game & Matchups

**Decision:** FIXED_PARTNERS uses persistent Session Teams. The host configures the teams before gameplay. PickleHub deterministically generates the initial team-vs-team matchup and court assignment. The host may review and adjust the initial matchup/court assignment before the session starts.

**Once gameplay begins:**

- Teams are locked; partners never change.
- Subsequent matchups are generated deterministically by the system.
- Matchups are team-vs-team.
- Court assignment is part of Game generation.
- Matchups/courts may rotate between games.
- Smart Rotation does not control Fixed Partners.
- No randomness. No LLM/AI.
- The same Game → Result → Confirmation → Stats pipeline is reused.

**Deferred:** exact matchup optimization algorithm.

### U7 — Tournament Mode

**Decision:** TOURNAMENT remains a recognized future Playing Session gameplay mode but is not enabled or implemented in MVP.

- MVP gameplay modes: SMART_ROTATION, FIXED_PARTNERS.
- Future: TOURNAMENT, integrated with a separate Tournament Engine when tournament functionality is implemented.
- Tournament mode is not deleted from the architecture; it is prevented from expanding MVP scope.
- No tournament-specific database/workflow requirements are added to MVP unless another already-approved feature requires them.

### U8 — Recurring Open Play

**Decision:** Recurring Open Play uses a separate Recurring Open Play Schedule that generates independent Open Play instances. The schedule is the template; each generated instance is an independent operational event.

**Each instance owns:** date/time, lifecycle, registrations, check-ins, courts, Playing Session, Games, Results, cancellation state.

**Rules:**

- Players register for an individual Open Play instance.
- Individual occurrences can be cancelled or adjusted.
- Changing the recurring schedule does not rewrite historical occurrences.
- Deactivating the recurring schedule prevents future generation.
- Existing instances are retained.
- Generated instances become independent operational records.
- Recurrence generation is deterministic.

**Deferred:** exact generation horizon, advanced recurrence syntax, complex recurrence patterns. (Further defined by B2.)

### U9 — Open Play Skill Level

**Decision:** Open Play has a skill-level classification used for informational/discovery purposes.

**MVP values:** BEGINNER, INTERMEDIATE, ADVANCED, ALL_LEVELS.

**Skill level does not:** restrict registration, restrict check-in, determine Smart Rotation, act as an authorization rule, or act as an ELO/DUPR/rating system. It helps players determine whether an Open Play is suitable for them.

Open Play skill level and player skill level are separate concepts. Future versions may introduce stricter skill-based sessions if needed.

### U10 — Booking Participants & Invitations

**Decision:** Registered players join private bookings through an invitation/acceptance workflow. The Booking Owner sends an invitation to a registered user.

```text
Invitation: PENDING → ACCEPTED | DECLINED | CANCELLED
```

**Rules:**

- Only an accepted invitation creates an active Booking Participant.
- Declining does not create participation.
- Cancelled invitations do not create participation.
- Invitation ≠ participation.
- Server validates invitation acceptance.
- The notification system can notify invited players.
- Invitation records are retained historically.

**Guests** are added directly by the Booking Owner (no account, no invitation workflow). Guests can participate in private bookings and cannot own bookings.

**Deferred:** invitation expiration rules. (Booking Owner participation is defined by I7.)

### U11 — Booking Fees & Payments

**Decision:** Payment processing is excluded from the PickleHub MVP. Court bookings are confirmed after successful booking validation and are not dependent on payment status. Fee collection is handled outside PickleHub during MVP.

**Rules:**

- No payment gateway in MVP.
- No payment-required booking state.
- Booking lifecycle remains CONFIRMED, CANCELLED, COMPLETED.
- Payment does not control Booking authority in MVP.
- No payment data is required in the MVP schema unless another approved feature requires it.
- Future payment functionality is a separate Payment domain associated with Booking (online payment, refunds, payment tracking).

**Core principle:** Booking and payment remain separate domains. (Further defined by I12.)

### U12 — Check-In States & Session Participant Leaving

**Decision:** Check-In = Open Play attendance. Session Participant = current Playing Session participation and future gameplay eligibility.

```text
Check-In:            CHECKED_IN, CHECKED_OUT
Session Participant: ACTIVE, LEFT
Flow:                Registration → Check-In → Session Participant → Game Participant
```

**Rules:**

- Registration = intent to attend. Check-In = attendance. Session Participant = current session participation. Game Participant = actual Game assignment.
- A player who leaves becomes LEFT; LEFT excludes the player from future game generation.
- Previous Games remain unchanged.
- Checking out does not automatically terminate an active Game.
- Historical attendance and session participation remain preserved.
- Check-In and Session Participant do not duplicate each other's authority.

**Deferred:** rejoining after leaving; exact handling of unexpected mid-game departure; whether checkout is mandatory at session completion. (Further defined by I3.)

### U13 — Open Play Cancellation State

**Decision:** Open Play includes CANCELLED as a terminal lifecycle state.

```text
SCHEDULED → REGISTRATION_OPEN → SESSION_READY → ACTIVE → COMPLETED

SCHEDULED         → CANCELLED
REGISTRATION_OPEN → CANCELLED
SESSION_READY     → CANCELLED
```

**Rules:**

- CANCELLED is terminal.
- A cancelled Open Play cannot start or create a Playing Session.
- Existing registrations remain historically recorded.
- Cancellation notifications can be sent; audit/history is retained.
- An ACTIVE Open Play is not changed to CANCELLED; early termination of an active session is handled through Playing Session/session-management rules.
- Cancelling one recurring occurrence does not cancel the recurring schedule; other instances remain unaffected.

(The SESSION_READY → COMPLETED path for insufficient players is defined by I4(c).)

### U14 — Player + Facility Admin Capabilities

**Decision:** One PickleHub account may hold both PLAYER and FACILITY_ADMIN. FACILITY_ADMIN does not exclude normal player participation.

- **As a player:** register for Open Play, check in, participate in Playing Sessions, play Games, receive Player Statistics.
- **As an administrator:** manage facility functionality, courts, and Open Play; resolve disputes (subject to I1); perform authorized administrative actions.
- SESSION_HOST remains contextual, not a global account role.

**Security rules:** players cannot self-promote to FACILITY_ADMIN; administrative authority must be granted through an authorized process; server-side authorization remains authoritative; exact physical role/capability representation is deferred to schema design.

### U15 — Guest Game Result Disputes

**Decision:** Guests cannot directly submit formal Game Result disputes in MVP.

```text
Guest → raises concern → Session Host → formal Dispute → FACILITY_ADMIN → review / resolution
```

**Rules:**

- Registered players can submit formal disputes.
- Guests cannot directly create formal Dispute records.
- The Session Host can submit a formal dispute on the guest's behalf.
- Facility Admin resolves the dispute.
- Guest participation remains preserved through Game Participant records.
- Disputed Results remain excluded from official statistics until resolved.
- No guest-specific authentication/dispute workflow is required for MVP.

### U16 — Correcting Wrong Players or Teams

**Decision:** Incorrect Game Team or Participant assignments are not corrected through destructive editing. The original assignment is preserved, and a Facility Admin can create an authoritative correction through an auditable correction mechanism.

**Rules:**

- Do not destructively edit historical Game Participants; do not delete or recreate the Game.
- The original participant/team assignment is preserved; the corrected assignment becomes authoritative.
- Facility Admin performs the correction; the correction is recorded in audit history.
- Game remains COMPLETED.
- Game Result remains unchanged unless it separately requires correction.
- Statistics use the corrected authoritative Game participation and authoritative Result.
- Later Games are not automatically deleted, regenerated, or rescheduled.
- Players and Session Hosts cannot directly rewrite historical Game Participants after confirmation.

**Deferred:** exact physical database representation of participant/team corrections.

### U17 — Session Host Confirming Their Own Game

**Decision:** The Session Host may submit and confirm the final Game Result even when they personally participated in that Game. Another player is not required for normal confirmation. Confirmed Results can still be disputed; the host cannot resolve their own dispute; U1–U4 rules remain unchanged.

**Core principle:** Host confirmation is trusted for normal gameplay, with post-confirmation dispute and administrative correction providing the safeguard.

### U18 — Active Game at Booking End

**Decision:** The Booking `end_time` is a hard reservation boundary for a private Playing Session.

**Rules:**

- The Playing Session cannot extend beyond the Booking.
- The system should prevent starting a new Game when the remaining Booking window is insufficient.
- No new Game can start after the Booking boundary.
- PickleHub must not automatically invent a Game Result, declare a winner, or silently extend the Booking.
- An active Game reaching the Booking end remains unresolved; historical gameplay is preserved.
- An unresolved active Game prevents normal Session completion and must be handled through an authorized session-management process.

**Deferred:** exact administrative handling of an overrun Game.

### U19 — Open Play Host Designation

**Decision:** Open Play is facility-controlled. A FACILITY_ADMIN creates and manages the Open Play and may designate an authorized FACILITY_ADMIN as the Open Play Host.

```text
FACILITY_ADMIN → creates/manages → OPEN PLAY → designates → OPEN PLAY HOST → PLAYING SESSION
```

**Rules:**

- Open Play is created and managed by the facility (FACILITY_ADMIN).
- Open Play Host is contextual and must be an authorized FACILITY_ADMIN in MVP.
- The creator and host do not have to be the same person.
- Open Play Host receives normal Session Host gameplay authority: start/manage the Playing Session; submit and confirm Game Results per U5/U17.
- Host cannot resolve disputes (see I1).
- Host does not gain global administrative permissions.
- Facility Admin can change the designated host before gameplay; host changes after gameplay begins require administrative intervention.
- Open Play participants do not automatically become hosts.

### B1 — Submitted Result Correction

**Decision:** Every correction or resubmission creates a new Game Result version. The previous Result becomes SUPERSEDED. No Game Result is overwritten. This applies even when the Result is still SUBMITTED and has not yet been confirmed.

```text
SUBMITTED
   ├──→ CONFIRMED
   └──→ SUPERSEDED

CONFIRMED ──→ SUPERSEDED
```

A new Result version may then be created.

### B2 — Recurring Open Play Schedule

**Decision:** Recurring Open Play uses a separate Recurring Open Play Schedule.

**MVP recurrence:**

- weekly by selected day of week
- start time
- end time
- start date
- optional end date
- facility timezone

**Schedule template information includes:**

- title/name
- description
- recurrence configuration
- skill level
- capacity
- registration timing
- informational fee
- status
- created/updated information

**Schedule status:** ACTIVE, INACTIVE.

**Generated instances:**

- Each generated Open Play instance is independent and retains a reference to its originating schedule.
- Generated instances own their date/time, lifecycle, registrations, check-ins, court reservations, Playing Session, Games, Results, and cancellation state.
- Deactivating a schedule prevents future generation but does not modify existing instances.

**Validation:** Generated instances must be validated against facility operating hours, court availability, bookings, court blockouts, and facility closures. The system must not silently create conflicting reservations, silently change time/court, or delete an occurrence. Conflicting generated occurrences require facility-admin review and may be adjusted or cancelled independently.

**Deferred:** advanced recurrence syntax and complex recurrence patterns.

### I1 — Admin / Host Conflict of Interest

**Decision:** A FACILITY_ADMIN may resolve disputes unless that admin is:

- the Session Host,
- the Open Play Host, or
- a participant in the disputed Game.

A conflicted administrator cannot resolve their own hosted/played dispute. Another eligible FACILITY_ADMIN must resolve it. If no eligible administrator is available, the dispute remains unresolved until an eligible administrator can review it.

### I3 — Check-Out vs LEFT

**Decision:**

- `Session Participant.status` is the sole authority for future gameplay eligibility.
- CHECKED_IN / CHECKED_OUT = attendance.
- ACTIVE / LEFT = session participation.
- Checking out causes the Session Participant to become LEFT.
- LEFT excludes the player from future Game generation.
- Checking out does not terminate an active Game.

### I4(c) — Open Play With Fewer Than 4 Players

**Decision:** If an Open Play reaches SESSION_READY with fewer than 4 eligible checked-in players:

- gameplay cannot start
- no Playing Session is created
- no Games are created
- no game statistics are created
- registration/check-in history is preserved
- the Open Play may transition to COMPLETED

This is distinct from CANCELLED. CANCELLED means the facility cancelled the Open Play before activation.

### I5/I6 — Authorized Scorer Designation and Confirmation

**Decision:** Authorized Scorers:

- are designated at the Playing Session level
- must be registered users
- do not need to participate
- have authority only for that session

The Session Host or a FACILITY_ADMIN may designate/remove them.

| Role | Submit final score | Confirm Game completion | Resolve disputes |
|---|---|---|---|
| Authorized Scorer | CAN | CANNOT (MVP) | CANNOT |
| Session Host | CAN | CAN | CANNOT |
| FACILITY_ADMIN | CAN | CAN | CAN (subject to I1) |

### I7 — Booking Owner Automatically Participates

**Decision:**

- The registered Booking Owner is automatically created as an active Booking Participant when a private Booking is created.
- No invitation is required for the owner.
- Other registered players require accepted invitations.
- Guests can be directly added by the Booking Owner.
- `Booking.owner_id` remains the ownership authority.

### I8 — No Automatic Result Timeout in MVP

**Decision:** There is no automatic Game Result timeout or automatic confirmation in MVP. A SUBMITTED Result remains SUBMITTED until the Session Host confirms or a FACILITY_ADMIN intervenes.

The system must not automatically confirm the Result, declare a winner, complete the Game, or generate the next Game.

**Deferred:** future timeout/reminder/escalation behavior.

### I12 — Fees Are Informational Only

**Decision:** Open Play may display an informational fee. The fee is not a payment transaction, does not create payment state, does not require payment, and does not involve payment processing.

- No payment processing, payment verification, reconciliation, refunds, or payment-dependent booking state in MVP.
- Court booking payment functionality is also excluded from MVP.
- Future payments are a separate Payment domain.

### I14 — Guest Identity Scope

**Decision:**

- Guest Identity is private-context scoped in MVP and is associated with the relevant private Booking context.
- It is NOT a global player identity.
- The same display name in another booking does not imply the same person.
- No automatic guest identity merging.
- Guests do not receive persistent global Player Statistics.
- Future guest-to-account claiming requires explicit identity verification.

---

## 11. Supersession Map

| Original Step 5 rule (section 3/4/5) | Current approved rule | Decision |
|---|---|---|
| Result lifecycle `ACTIVE → RESULT_SUBMITTED → CONFIRMED` (5.6) | Game: SCHEDULED → ACTIVE → COMPLETED; Result: SUBMITTED → CONFIRMED / SUPERSEDED | U2, U3, B1 |
| Dispute path `CONFIRMED → DISPUTED → …` as a Result path (5.6) | Dispute is a separate entity: DISPUTED → ADMIN_REVIEW → RESOLVED | U2, U3 |
| Game statuses include RESULT_SUBMITTED, CONFIRMED, DISPUTED (5.20) | Game statuses: SCHEDULED, ACTIVE, COMPLETED | U2, U3, U4 |
| "One current Result per Game" (5.23, section 5) | Multiple Result versions; one authoritative | U1, B1 |
| `dispute_id` on Game Result (5.23) | Removed; Dispute references Result | U1 |
| Game → Game Result (0..1); Result → Dispute (0..1) (4.7) | 0..N Result versions (one authoritative); 0..N Disputes (max one active) | U1 |
| Result may be overwritten when corrected (implicit) | New version; previous SUPERSEDED, including from SUBMITTED | U1, B1 |
| Three private gameplay modes (5.8) | MVP: SMART_ROTATION, FIXED_PARTNERS; TOURNAMENT future | U7 |
| "Only game participants can submit normal disputes" (5.24) | Registered participants; host submits on a guest's behalf | U15 |
| "Facility Admin resolves" (5.24) | Facility Admin resolves, excluding conflicted admins | I1 |
| Open Play statuses without CANCELLED (5.25) | CANCELLED is a terminal state | U13 |
| "Zero checked-in players may end without games" (5.3, 5.25, 5.37) | Fewer than 4 at SESSION_READY: no session; may transition to COMPLETED | I4(c) |
| "facility admin/staff/designated Open Play host" (5.3, 5.4, 5.18, 5.25, 5.28) | FACILITY_ADMIN; Open Play Host must be FACILITY_ADMIN | U14, U19 |
| Check-In `status` values undefined; eligibility considers check-out (5.28) | CHECKED_IN / CHECKED_OUT; Session Participant status is sole eligibility authority | U12, I3 |
| Booking Participant joins without invitation concept (5.17) | Invitations for registered players; owner auto-participant; guests direct | U10, I7 |
| Open Play `fee` without defined meaning (5.25) | Informational only; no payment state | U11, I12 |
| Recurring Open Play "creates separate instances" (5.3, 5.25) | Recurring Open Play Schedule entity with defined MVP recurrence | U8, B2 |
| Guest Identity "scoped to the relevant private context" (5.10) | Associated with the private Booking context; not global | I14 |

---

## 12. Remaining Non-Blocking Items

These items were identified during the consistency audits, are **not** approved decisions, and do not block physical schema design. They must not be resolved by assumption; they are to be confirmed during the relevant schema-design or feature step.

| ID | Item | Where it is resolved |
|---|---|---|
| I2 | Under U16, whether a corrected Game Participant must be a Session Participant of the same session, and whether Smart Rotation uses corrected or original history | Schema design for Game Participant corrections |
| I4(a)(b) | Terminal handling of an overrun Game (U18, deferred) and of a Playing Session that is created but never started | Session-management feature; status representation should allow additive states |
| I9 | Conceptual fields of Session Team and Session Team Member (membership status/end, stable team identifier) — derivable from 5.35 and U6 | Schema design |
| I10 | `Open Play.host_id` vs `Playing Session.host_id` — which is authoritative before/after session creation | Schema design |
| I11 | Representation of "flagged/reviewed" conflicts (facility closures, court blockouts, B2 generated occurrences) | Schema design |
| I13 | MVP Player Profile fields and profile visibility controls (Step 1 §23) | Profile feature / schema design |
| I15 | Definition of "Court Sessions Played" vs "Open Play Sessions Attended" | Statistics feature |
| I16 | Facility Closure `reason_type` values, status, and `created_by`; Operating Hours "active" flag; invitation/correction notification and audit type names; who may cancel a booking | Schema design |
| — | Events, Community (connections, groups) are not covered by Step 5 decisions | Later feature steps |

---

Status: STEP 5 DECISIONS FINALIZED — READY FOR PHYSICAL SCHEMA DESIGN

Previous status: STEP 5 DECISIONS RECORDED — READY FOR CONSISTENCY AUDIT

No SQL schema or migrations have been created.
