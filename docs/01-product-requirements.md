# PickleHub — Step 1: Product Definition & Requirements

> This is the finalized Step 1 product specification for PickleHub (Version 1.0).
> It is the primary source of product requirements for the project.
> The original Step 1 text below is preserved unchanged. Approved later amendments are recorded in the "Post-Step-1 Amendments" section at the end of this document and take precedence where they differ. Full decision details are in `docs/03-step-5-database-decisions.md`.

**Project Type:** Pickleball Facility Management & Community Platform\
**Working Name:** PickleHub\
**Project Status:** Product Definition\
**Version:** 1.0

## 1. Product Concept

PickleHub is a modern web platform designed for a **specific pickleball
facility**.

It is not a marketplace for discovering pickleball courts around
different locations. The website represents and manages **our own
pickleball facility and courts**.

The platform allows players to:

-   Learn about pickleball

-   View the facility's courts

-   Check court availability

-   Book courts

-   Invite other players to bookings

-   Join public Open Play sessions

-   Participate in events and tournaments

-   Create personal player profiles

-   Track games, wins, losses, and statistics

-   Connect with other players

-   Eventually use an AI Pickleball Assistant and Coach

The central idea is:

**A digital home for players who want to play, connect, and improve at
our pickleball facility.**

## 2. Core Product Philosophy

The product should revolve around the actual experience of playing
pickleball.

The primary journey is:

PLAYER

↓

BOOK A COURT or JOIN OPEN PLAY

↓

CREATE / JOIN PLAYING SESSION

↓

PLAY GAMES

↓

RECORD RESULT

↓

UPDATE PLAYER STATISTICS

↓

SMART PLAYER ROTATION

↓

PLAY NEXT GAME

↓

CONNECT WITH PLAYERS

↓

IMPROVE SKILLS

The website should therefore not feel like a generic sports information
website.

It should feel like a **real digital system for operating and
participating in a pickleball facility**.

## 3. Target Users

### 3.1 Beginner Players

People who are new to pickleball.

They can:

-   Learn what pickleball is

-   Learn the rules

-   Learn scoring

-   Learn how to serve

-   Learn court positioning

-   Learn basic strategies

-   Learn pickleball terminology

-   Learn about equipment

-   Join beginner-friendly Open Play

-   Track their games

-   Eventually receive AI coaching

### 3.2 Regular Players

Players who already know how to play.

They can:

-   Book courts

-   Join Open Play

-   Participate in events

-   Create player profiles

-   Track games

-   Track wins and losses

-   View statistics

-   Connect with other players

-   View previous games

-   Improve their skills using AI tools

### 3.3 Facility Administrators

The facility owner or staff members.

They can eventually:

-   Manage courts

-   Set court availability

-   Block courts

-   Manage bookings

-   Create recurring Open Play sessions

-   Manage Open Play capacity

-   Manage events

-   Manage tournaments

-   Manage players

-   Manage facility announcements

-   View operational statistics

-   Manage session rules

-   Resolve disputed game results

### 3.4 Tournament Organizers

This will be a later-stage user type.

They can:

-   Create tournaments

-   Create divisions

-   Register players

-   Register teams

-   Generate brackets

-   Schedule matches

-   Record scores

-   Display standings

-   Display tournament results

## 4. Main Product Areas

PickleHub will eventually consist of these major areas:

PICKLEHUB

│

├── Home

│

├── Learn

│ ├── Pickleball Basics

│ ├── Rules

│ ├── Scoring

│ ├── How to Play

│ ├── Strategies

│ └── Equipment

│

├── Our Courts

│ ├── Courts

│ └── Court Details

│

├── Book a Court

│

├── Open Play

│ ├── Upcoming Sessions

│ ├── Session Details

│ └── My Registrations

│

├── Events

│

├── Tournaments

│

├── Community

│ ├── Players

│ ├── Groups

│ └── Connections

│

├── AI Coach

│

├── About

│

└── Account

├── Dashboard

├── Profile

├── Bookings

├── Open Play

├── Games

└── Settings

Not all areas will be built in the MVP.

## 5. Learn Pickleball

The Learn section is the educational knowledge base of PickleHub.

It should contain:

-   What is pickleball?

-   Basic rules

-   Scoring

-   Serving

-   Court dimensions

-   Kitchen / Non-Volley Zone

-   Doubles rules

-   Singles rules

-   Dinking

-   Volleys

-   Third-shot drop

-   Groundstrokes

-   Court positioning

-   Beginner strategies

-   Intermediate strategies

-   Pickleball terminology

-   Equipment basics

-   Paddle information

-   Ball information

-   Shoes and accessories

This content will eventually also provide a knowledge base for the AI
Pickleball Assistant.

## 6. Our Courts

PickleHub represents a facility with its own courts.

This is different from a general court-discovery platform.

For example:

OUR COURTS

Court 1

Available

Court 2

Booked

Court 3

Available

Court 4

Maintenance

Each court can have:

-   Court name/number

-   Status

-   Indoor/outdoor

-   Description

-   Photos

-   Availability

-   Operating hours

-   Booking rules

## 7. Court Booking

Players can book one of the facility's courts.

Basic flow:

PLAYER

↓

Book a Court

↓

Choose Date

↓

Choose Time

↓

View Available Courts

↓

Select Court

↓

Confirm Booking

↓

Invite/Add Players

↓

Booking Created

Example:

Court 1

October 10, 2026

9:00 AM -- 12:00 PM

Booked by:

Juffy

Players:

Juffy

Mark

Sarah

John

Anna

Mike

The booking can then become a **Playing Session**.

## 8. Playing Sessions

A major architectural concept of PickleHub is the **Playing Session**.

A Playing Session represents an actual group of players playing games on
a court.

There are initially two types:

PLAYING SESSION

│

├── BOOKING SESSION

│

└── OPEN PLAY SESSION

Eventually:

PLAYING SESSION

│

├── BOOKING

├── OPEN PLAY

└── TOURNAMENT

This allows the same game and rotation engine to be reused across
different types of play.

## 9. Court Booking → Playing Session

Example:

Six players book Court 1 from 9 AM--12 PM.

Booking

9:00 AM -- 12:00 PM

Players:

Juffy

Mark

Sarah

John

Anna

Mike

The booking owner can select:

START PLAYING SESSION

The system then creates:

Playing Session

│

├── Court 1

├── 9:00 AM -- 12:00 PM

├── 6 players

└── Doubles

The system can then automatically generate the first game.

## 10. Open Play

Open Play is a core feature.

**Definition**

Open Play is a scheduled public session where players register
individually and play rotating games with other registered participants.

Players do not need to bring a specific partner.

Example:

OPEN PLAY

Every Saturday

1:00 PM -- 4:00 PM

Court:

Court 1

Type:

Public Open Play

Skill Level:

Beginner / Intermediate

Capacity:

12 Players

Fee:

₱150

\[JOIN OPEN PLAY\]

## 11. Recurring Open Play

The facility administrator can create a recurring Open Play schedule.

Example:

Day:

Saturday

Time:

1:00 PM -- 4:00 PM

Court:

Court 1

Type:

Public Open Play

Capacity:

12

Recurrence:

Every Saturday

The system can automatically generate individual sessions:

Saturday, October 3

1:00 PM -- 4:00 PM

8 / 12 players

Saturday, October 10

1:00 PM -- 4:00 PM

4 / 12 players

Saturday, October 17

1:00 PM -- 4:00 PM

10 / 12 players

## 12. Open Play Registration

Players can register individually.

Example:

Saturday Open Play

October 3

1:00 PM -- 4:00 PM

Court 1

8 / 12 Players

₱150

\[JOIN OPEN PLAY\]

After registration:

You're registered!

Saturday

1:00 PM -- 4:00 PM

Players:

8 / 12

\[View Session\]

\[Cancel Registration\]

## 13. Open Play Check-In

Registration does not automatically mean the player is present.

The system should distinguish:

Registered Players

↓

Checked-In Players

↓

Playing Players

Example:

Registered:

12

Checked In:

9

Only checked-in players are included in the actual game rotation.

This prevents absent players from being selected by the rotation system.

## 14. Open Play Game Rotation

Suppose six players check in:

Juffy

Mark

Sarah

John

Anna

Mike

The system knows:

Players = 6

Players per game = 4

Format = Doubles

The system generates the first game automatically.

Example:

GAME 1

Team A

Juffy

Sarah

VS

Team B

Mark

Anna

Waiting:

John

Mike

The players play the game.

## 15. Result Recording

After the game finishes, the players record the result.

Example:

GAME 1

Juffy + Sarah

11

Mark + Anna

8

\[SUBMIT RESULT\]

The next game must NOT be generated yet.

The system waits for the result.

## 16. Result Confirmation

To reduce incorrect or fake results, another player can confirm the
result.

Example:

Result Submitted

Juffy + Sarah

11

Mark + Anna

8

Waiting for confirmation\...

Juffy ✓

Sarah ✓

Mark ✓

Anna ✓

After the required confirmation:

RESULT CONFIRMED ✓

Updating player statistics\...

Generating next rotation\...

A disputed result can be flagged for facility administration.

## 17. Core Rotation Rule

This is a fundamental PickleHub business rule:

**The next game must not be generated until the previous game has a
confirmed result.**

The workflow is:

GAME 1

↓

Players play

↓

Submit result

↓

Confirm result

↓

Update statistics

↓

Calculate next rotation

↓

GAME 2

↓

Players play

↓

Submit result

↓

Confirm result

↓

Calculate next rotation

This continues until the session ends.

## 18. Smart Rotation System

The system should not simply randomly shuffle players.

It should attempt to provide a fair and varied rotation.

The rotation algorithm should consider:

1.  Previous partners

2.  Previous opponents

3.  Number of games played

4.  Number of games waited

5.  Waiting time

6.  Wins/losses

7.  Current player availability

The primary goal is:

**Create varied games while fairly distributing playing and waiting
time.**

## 19. Example: Six Players

Players:

A

B

C

D

E

F

**Game 1**

A + B

VS

C + D

Waiting:

E + F

Result:

A + B = 11

C + D = 8

The system updates the statistics.

Then it generates Game 2.

**Game 2**

A + E

VS

B + F

Waiting:

C + D

The algorithm attempts to avoid:

A + B

being partners again.

It also considers who has been waiting.

## 20. Different Player Counts

The system should support different numbers of players.

**4 players**

4 Players

↓

4 Play

↓

0 Waiting

**5 players**

5 Players

↓

4 Play

↓

1 Waiting

**6 players**

6 Players

↓

4 Play

↓

2 Waiting

**7 players**

7 Players

↓

4 Play

↓

3 Waiting

**8 players**

8 Players

↓

4 Play

↓

4 Waiting

And so on.

The system should always attempt to prioritize players who have played
fewer games or waited longer.

## 21. Player Game Tracking

Every confirmed game contributes to a player's statistics.

Example:

Juffy

Games Played: 10

Wins: 6

Losses: 4

Win Rate: 60%

Additional statistics can eventually include:

-   Games played

-   Wins

-   Losses

-   Win rate

-   Points scored

-   Points against

-   Point differential

-   Open Play sessions attended

-   Court sessions played

-   Partners played with

-   Opponents played

-   Recent game results

## 22. Player Dashboard

Example:

WELCOME BACK, JUFFY

Upcoming

Saturday Open Play

1:00 PM

Court Booking

Tuesday

7:00 PM

────────────────────

YOUR STATS

Games Played

37

Wins

22

Losses

15

Win Rate

59.5%

────────────────────

RECENT GAMES

Oct 1

Win 11--8

Sep 29

Loss 7--11

Sep 27

Win 11--6

## 23. Player Profiles

Players can have a pickleball-specific profile.

Example:

Juffy Jhon

Intermediate

Doubles

37 Games

22 Wins

15 Losses

Win Rate: 59.5%

Open Plays:

12

Connections:

8

Players can control what profile information is visible to other users.

## 24. Community

The community system should not become a generic social-media platform.

Its purpose is to help players connect through actual pickleball
activities.

The primary community flow is:

JOIN OPEN PLAY

↓

MEET PLAYERS

↓

PLAY GAMES

↓

SEE OTHER PLAYERS

↓

CONNECT

↓

PLAY AGAIN

## 25. Player Connections

Players can optionally connect with other players.

Example:

Sarah

Intermediate

Doubles

\[Connect\]

Once connected:

My Connections

Sarah

Mark

John

Anna

The system can eventually allow players to see:

-   Shared Open Plays

-   Previous games

-   Favorite partners

-   Upcoming sessions

Privacy controls should determine what information is visible.

## 26. Groups

Groups are an optional future feature.

Instead of creating complicated clubs, PickleHub can support simple
player groups.

Examples:

Beginner Players

18 Members

Intermediate Players

32 Members

Weekend Players

27 Members

Morning Players

15 Members

Groups can eventually have:

-   Members

-   Group Open Play sessions

-   Events

-   Announcements

-   Discussions

This is not required for the MVP.

## 27. Session Host

For private court bookings, one player is designated as the Session
Host.

Example:

COURT BOOKING

Host:

Juffy

Players:

Juffy

Mark

Sarah

John

Anna

Mike

The host can:

-   Start the session

-   Manage players

-   Start the first game

-   End the session

-   Handle basic session management

The host does NOT need to manually organize every game.

The rotation engine handles that.

## 28. Open Play vs Court Booking

These are different products within PickleHub.

| Feature           | Court Booking  | Open Play                 |
| ----------------- | -------------- | ------------------------- |
| Created by        | Player         | Facility                  |
| Players           | Private/group  | Public participants       |
| Partner required  | Usually        | No                        |
| Court             | Reserved court | Facility-designated court |
| Payment           | Booking fee    | Open Play fee             |
| Capacity          | Booking rules  | Session capacity          |
| Rotation          | Optional       | Automatic                 |
| Game tracking     | Yes            | Yes                       |
| Result recording  | Yes            | Yes                       |
| Player statistics | Yes            | Yes                       |

## 29. Game Scoring

Initial scoring configuration:

Game to:

11

Win by:

2 points

Valid examples:

11--8 ✓

11--9 ✓

12--10 ✓

Invalid:

11--10 ✗

The facility should eventually be able to configure:

Game To:

11 / 15 / 21

Win By 2:

Yes / No

## 30. Smart Rotation Engine Architecture

The rotation engine should be a reusable application module.

PLAYING SESSION

│

┌─────────────┴─────────────┐

│ │

BOOKING OPEN PLAY

│ │

└─────────────┬─────────────┘

↓

ROTATION ENGINE

│

┌─────────────┼─────────────┐

↓ ↓ ↓

Previous Games Played Waiting Time

Partners

│ │ │

└─────────────┼─────────────┘

↓

TEAM GENERATION

↓

NEXT GAME

Eventually tournaments can use the same game/session infrastructure
where appropriate.

## 31. Main Product Architecture

PICKLEHUB

│

┌─────────────────────┼─────────────────────┐

│ │ │

LEARN PLAY EVENTS

│ │ │

│ ┌───────┴────────┐ │

│ │ │ │

│ BOOKING OPEN PLAY │

│ │ │ │

│ └───────┬────────┘ │

│ │ │

│ PLAY SESSION │

│ │ │

│ SMART ROTATION │

│ │ │

│ GAMES │

│ │ │

│ RESULTS │

│ │ │

│ PLAYER STATISTICS │

│ │ │

└─────────────────────┼─────────────────────┘

│

PLAYER ACCOUNT

│

┌──────────────┼──────────────┐

│ │ │

Profile Games Connections

│ │

└──────────────┼──────────────┘

│

▼

AI PICKLEBOT

│

┌──────────┼──────────┐

│ │ │

Q&A Coach Training

## 32. Future AI Pickleball Assistant

The AI assistant will eventually be called something like:

**PickleBot**

It can answer questions such as:

\"How does pickleball scoring work?\"

\"What is a dink?\"

\"What's the difference between a volley and a groundstroke?\"

\"I'm a beginner. What should I practice?\"

\"What paddle should a beginner use?\"

The AI can use PickleHub's own pickleball knowledge base.

## 33. AI Coaching

Eventually:

AI PICKLEBALL COACH

What's your goal?

○ Beginner

○ Improve consistency

○ Improve serving

○ Improve dinking

○ Improve doubles strategy

Practice frequency:

3 days/week

\[GENERATE TRAINING PLAN\]

Example output:

4-WEEK TRAINING PLAN

Week 1

Paddle control

Serving

Basic positioning

Week 2

Dinking

Volleys

Week 3

Third-shot drops

Transition zone

Week 4

Game strategy

Match practice

## 34. AI and Player Statistics

Eventually, the AI could use a player's own statistics.

Example:

Juffy

Games: 37

Wins: 22

Losses: 15

Win Rate: 59.5%

The AI could eventually provide insights such as:

-   Areas to practice

-   Training recommendations

-   Suggested drills

-   Progress summaries

AI should not directly control the core booking or scoring logic.

The booking, game, scoring, and rotation systems should remain
deterministic application logic.

## 35. Tournament System

Future functionality:

TOURNAMENT

│

├── Registration

├── Players

├── Teams

├── Categories

├── Brackets

├── Matches

├── Scores

├── Standings

└── Results

Example:

PICKLEHUB AUTUMN OPEN

October 24--25

Men's Doubles

Women's Doubles

Mixed Doubles

\[REGISTER\]

## 36. Admin Dashboard

The facility needs its own management interface.

Example:

ADMIN DASHBOARD

Today's Overview

Bookings 24

Open Play 3

Events 2

Revenue ₱8,450

────────────────────

COURTS

Court 1 Available

Court 2 Booked

Court 3 Available

Court 4 Maintenance

────────────────────

UPCOMING

Open Play

6:00 PM

Tournament

Saturday

Admin functionality will eventually include:

-   Court management

-   Booking management

-   Open Play management

-   Player management

-   Event management

-   Tournament management

-   Session management

-   Result dispute management

-   Facility settings

-   Analytics

## 37. MVP Scope

The first version should NOT attempt to build everything.

**Public Website**

-   Home

-   About

-   Learn

-   Our Courts

-   Open Play

-   Events

-   Contact

**Booking**

-   Court availability

-   Court selection

-   Date/time selection

-   Booking creation

-   Player invitations

**Open Play**

-   Open Play schedules

-   Recurring Open Play

-   Registration

-   Capacity

-   Check-in

-   Playing session

**Accounts**

-   Registration/login

-   Player profile

-   My bookings

-   My Open Plays

**Game System**

-   Playing session

-   Automatic first-game generation

-   Game scoring

-   Result submission

-   Result confirmation

-   Player statistics

-   Automatic rotation

This is our **core MVP**.

## 38. Feature Roadmap

| Phase | Feature                   | Priority  |
| ----- | ------------------------- | --------- |
| 1     | Product definition        | Complete  |
| 2     | Development environment   | Core      |
| 3     | Claude Code configuration | Core      |
| 4     | Next.js project           | Core      |
| 5     | Design system             | Core      |
| 6     | Public website            | Core      |
| 7     | Court pages               | Core      |
| 8     | Court availability        | Core      |
| 9     | Booking system            | Core      |
| 10    | Open Play                 | Core      |
| 11    | User accounts             | Core      |
| 12    | Playing Sessions          | Core      |
| 13    | Game recording            | Core      |
| 14    | Smart Rotation            | Core      |
| 15    | Player statistics         | Core      |
| 16    | Player profiles           | Important |
| 17    | Player connections        | Important |
| 18    | Events                    | Important |
| 19    | Admin dashboard           | Important |
| 20    | Groups/community          | Future    |
| 21    | Tournaments               | Future    |
| 22    | AI Assistant              | Future    |
| 23    | AI Coach                  | Future    |
| 24    | AI Training Plans         | Future    |
| 25    | Production optimization   | Core      |
| 26    | Vercel deployment         | Core      |

## 39. Development Philosophy

PickleHub should be developed incrementally.

We should NOT tell Claude Code:

\"Build the entire PickleHub platform.\"

Instead:

Define

↓

Design

↓

Build small feature

↓

Test

↓

Review

↓

Commit

↓

Build next feature

Example:

Create project

↓

Create design system

↓

Build Navbar

↓

Build Homepage

↓

Build Courts

↓

Build Booking UI

↓

Build Open Play

↓

Build Accounts

↓

Build Playing Sessions

↓

Build Rotation Engine

↓

Build Statistics

## 40. Technology Direction

The planned stack is:

Frontend

Next.js

TypeScript

Tailwind CSS

Backend

Next.js server-side functionality

Database

Supabase / PostgreSQL

Authentication

Supabase Auth

Source Control

Git

GitHub

Development

VS Code

Claude Code

Deployment

Vercel

Future AI

AI API / LLM integration

The exact technology choices will be finalized during the technical
setup phase.

## 41. High-Level Technical Architecture

USER

│

▼

NEXT.JS APP

│

┌────────────┴────────────┐

│ │

FRONTEND BACKEND

│ │

│ API / Server

│ │

└────────────┬────────────┘

│

SUPABASE

│

PostgreSQL

│

┌────────────┼────────────┐

│ │ │

Users Bookings Games

│ │ │

│ Sessions Results

│ │ │

└────────────┼────────────┘

│

Rotation Engine

│

Player Stats

## 42. Core Data Relationship Concept

The eventual database will roughly follow:

USER

│

├── PROFILE

│

├── BOOKINGS

│ │

│ └── PLAYING SESSION

│ │

│ ├── PLAYERS

│ │

│ └── GAMES

│ │

│ ├── TEAMS

│ ├── SCORE

│ └── RESULT

│

└── OPEN PLAY REGISTRATIONS

│

└── PLAYING SESSION

│

└── GAMES

This is only the conceptual model for Step 1. Detailed database schema
design comes later.

## 43. Core Business Rules

The following rules are part of the initial product definition.

**Booking**

1.  A court cannot have conflicting confirmed bookings.

2.  A booking has a start and end time.

3.  A booking can have multiple players.

4.  A booking can become a Playing Session.

5.  A Session Host manages the session.

**Open Play**

1.  Open Play is created by the facility.

2.  Open Play can recur on a schedule.

3.  Players register individually.

4.  Open Play has a capacity.

5.  Registration does not equal attendance.

6.  Only checked-in players participate in rotation.

7.  Open Play does not require players to bring partners.

**Games**

1.  Games are played using configurable scoring rules.

2.  Players record the result.

3.  Results should be confirmed.

4.  A disputed result can be escalated to an administrator.

5.  Player statistics update after a confirmed result.

**Rotation**

1.  The first game is generated automatically.

2.  The next game is NOT generated until the current game has a
    confirmed result.

3.  The system should avoid repeated partners.

4.  The system should avoid unnecessary repeated opponents.

5.  Players who have played fewer games receive priority.

6.  Players who have waited longer receive priority.

7.  The system should distribute playing time fairly.

## 44. Final Product Flow

The complete future player experience:

PLAYER

│

▼

PICKLEHUB HOME

│

┌────────┴────────┐

│ │

BOOK COURT OPEN PLAY

│ │

│ REGISTER

│ │

│ CHECK-IN

│ │

└────────┬────────┘

│

▼

PLAYING SESSION

│

▼

GENERATE GAME 1

│

▼

PLAY GAME

│

▼

RECORD RESULT

│

▼

CONFIRM RESULT

│

▼

UPDATE STATISTICS

│

▼

SMART ROTATION

│

▼

NEXT GAME

│

▼

REPEAT

│

▼

SESSION ENDS

│

▼

PLAYER STATISTICS

│

▼

PLAYER COMMUNITY

│

▼

AI PICKLEBOT

## 45. Product Vision

The long-term PickleHub vision is:

**A complete digital platform for a pickleball facility where players
can learn, book, play, connect, track their progress, participate in
Open Play and events, and eventually receive personalized AI-powered
coaching.**

The product is centered around actual pickleball activity rather than
simply displaying information.

## 46. Step 1 Completion Criteria

Step 1 is considered complete when we agree on:

-   Product concept

-   Target users

-   Core features

-   Open Play behavior

-   Court booking behavior

-   Playing Sessions

-   Game recording

-   Result confirmation

-   Smart rotation

-   Player statistics

-   Community concept

-   Admin concept

-   AI direction

-   MVP scope

-   Future roadmap

-   High-level architecture

-   Core business rules

**Status: READY FOR TECHNICAL SETUP**

The next stage is **Step 2 --- Development Environment & Project
Setup**.

---

## Post-Step-1 Amendments

The following product amendments were approved during Step 5 (Database & Implementation Decisions). The original Step 1 sections above are preserved as the historical record; where they differ, these amendments take precedence. Decision IDs refer to `docs/03-step-5-database-decisions.md`.

### A1. Result Recording and Confirmation (supersedes the confirmation flow in §15, §16, §43)

- The Session Host (or an Authorized Scorer designated for the Playing Session) submits the final team scores. Only final scores are recorded.
- The Session Host confirms "Game Complete". Confirmation by every player is not required, and the host may confirm a game they played in (U5, U17).
- Authorized Scorers can submit scores but cannot confirm Game completion in MVP (U5, I5/I6).
- A FACILITY_ADMIN may submit and confirm when necessary (U5).
- There is no automatic confirmation or timeout; a submitted result waits for the Session Host or a FACILITY_ADMIN (I8).
- Every resubmission or correction creates a new result version; results are never overwritten (U1, B1).

### A2. Disputes (refines §16, §43)

- Registered players may dispute a confirmed result. Guests raise concerns through the Session Host, who may file the dispute on their behalf (U15).
- A FACILITY_ADMIN resolves disputes, excluding an admin who was the Session Host, the Open Play Host, or a participant in the disputed game (I1).
- A disputed result is excluded from official statistics until resolved. A dispute does not reopen the game, and later games are not regenerated (U2–U4).
- Incorrect player/team assignments are corrected by a FACILITY_ADMIN without destroying the original record (U16).

### A3. Private Booking Gameplay Modes (refines §28 "Rotation: Optional")

- Smart Rotation: partners and opponents rotate deterministically.
- Fixed Partners: partners stay together for the session; the system deterministically generates team-vs-team matchups and courts (U6).
- Tournament: a recognized future mode, not enabled in MVP (U7).
- Open Play always uses Smart Rotation.

### A4. Open Play (refines §10–§14, §28, §43)

- An Open Play may reserve one or more courts.
- Capacity is registration capacity. There is no waitlist in MVP.
- Each Open Play has an informational skill level: BEGINNER, INTERMEDIATE, ADVANCED, or ALL_LEVELS. It does not restrict registration, check-in, or rotation (U9).
- Open Play is started explicitly by the designated Open Play Host, who must be a FACILITY_ADMIN (U19). It does not start automatically at the scheduled time.
- Players may check themselves in; a FACILITY_ADMIN may correct attendance.
- If fewer than 4 eligible players are checked in when the Open Play is ready, gameplay does not start and the Open Play may be completed without games (I4(c)).
- The facility may cancel an Open Play before it becomes active (U13).
- No anonymous guests in Open Play in MVP.

### A5. Recurring Open Play (refines §11)

- A Recurring Open Play Schedule generates independent Open Play instances: weekly on a selected day of week, with start time, end time, start date, optional end date, in the facility timezone (U8, B2).
- Each instance is independent (registrations, attendance, gameplay, cancellation). Changing or deactivating a schedule does not change existing instances.
- Generated instances are validated against operating hours, court availability, bookings, court blockouts, and facility closures; conflicts require facility-admin review and are never silently resolved (B2).

### A6. Booking Participants and Invitations (refines §7, §9, §27)

- The Booking Owner automatically participates in the booking (I7).
- Other registered players join by invitation and must accept (U10).
- Guests without accounts may be added directly by the Booking Owner. Guests cannot own bookings, score, or file formal disputes, and do not receive persistent global statistics (U10, U5, U15, I14).

### A7. Fees and Payments (refines §10, §12, §28, §36)

- Payment processing is excluded from MVP. Booking confirmation does not depend on payment (U11).
- An Open Play may display an informational fee; it creates no payment state (I12).
- Future payments will be a separate Payment domain.

### A8. Roles

- One account may be both a Player and a Facility Admin (U14).
- "Facility Administrators: the facility owner or staff members" (§3.3) is represented by the FACILITY_ADMIN role in MVP; a separate Facility Staff role is deferred.
- Session Host, Open Play Host, and Authorized Scorer are contextual to a specific session, not global roles.
