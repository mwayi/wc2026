# World Cup Sweepstake

## Mission

Build a simple, beautiful World Cup 2026 sweepstake application that automatically calculates participant rankings from live World Cup match results.

The application should feel inspired by official World Cup tournament experiences while remaining clean, modern and easy to understand.

Primary objective:

* Every World Cup match should contribute to the sweepstake.
* Rankings should update automatically as results become available.
* Participants should be able to understand why they are winning or losing.

---

# Business Rules

## Team Allocation

Each participant receives:

* 1 Strong team
* 1 Mid-Level team
* 3 Weak or Very Weak teams

Teams should be distributed across groups where possible.

## Match Scoring

Each allocated team earns:

* Win = 3 points
* Draw = 1 point
* Loss = 0 points

## Participant Score

A participant's score is the aggregate score of all allocated teams.

Example:

Brazil Win = 3
Japan Draw = 1
Tunisia Loss = 0
Egypt Win = 3
Panama Loss = 0

Total = 7 points

## Ranking Order

Participants are ranked by:

1. Total Points
2. Goal Difference
3. Goals Scored

---

# Prize Distribution

Top three participants receive:

| Position | Prize |
| -------- | ----- |
| 1st      | 60%   |
| 2nd      | 30%   |
| 3rd      | 10%   |

If a child wins, the prize belongs to that child.

If an adult wins, proceeds may be donated to a good cause.

---

# Domain Model

## Country

Represents a World Cup nation.

```typescript
type Country = {
  id: string;
  fifaCode: string;
  name: string;

  group: string;

  pool:
    | 'strong'
    | 'mid'
    | 'weak'
    | 'veryWeak';

  aliases: string[];

  apiMappings: Record<string, string>;
};
```

Responsibilities:

* Stable internal identity.
* API mapping.
* Group ownership.
* Pool ownership.

Countries are the canonical source of truth for teams.

Never calculate scores directly from API team names.

---

## Participant

Represents a sweepstake player.

```typescript
type Participant = {
  id: string;
  name: string;
  countryIds: string[];
};
```

---

## Match

Represents a World Cup fixture.

```typescript
type Match = {
  id: string;

  homeCountryId: string;
  awayCountryId: string;

  homeGoals: number;
  awayGoals: number;

  status: string;
  date?: string;
};
```

---

## Participant Standing

Calculated projection.

```typescript
type ParticipantStanding = {
  participantId: string;

  points: number;

  goalsFor: number;
  goalsAgainst: number;

  goalDifference: number;

  played: number;
};
```

Do not persist standings.

Standings should always be derived.

---

# Data Flow

```text
World Cup Feed
    ↓
API Adapter
    ↓
Country Registry
    ↓
Match Entities
    ↓
Country Statistics
    ↓
Participant Statistics
    ↓
League Table
```

---

# Country Registry

The application owns a complete registry of World Cup teams.

Example:

```typescript
{
  id: "BRA",
  fifaCode: "BRA",
  name: "Brazil",

  aliases: [
    "Brazil"
  ],

  apiMappings: {
    apiFootball: "123",
    worldCupApi: "brazil"
  }
}
```

Benefits:

* Feed provider independence.
* Stable identifiers.
* Easier testing.
* Multiple feed support.

---

# API Integration

## Principle

External providers are not trusted as canonical models.

Every feed must be normalised into internal entities.

Example:

```text
External Feed
    ↓
Adapter
    ↓
Match
```

Never allow provider-specific structures to leak into application code.

---

# UI Requirements

## Dashboard

Contains:

* Tournament hero section
* Trophy branding
* Feed status
* Last updated timestamp
* Quick statistics

---

## League Table

Primary application view.

Displays:

* Rank
* Participant
* Points
* Goal Difference
* Goals For
* Goals Against
* Played
* Allocated Teams

Updates automatically as scores change.

---

## Participants

Displays:

* Participant
* Allocated teams
* Team categories
* Team contribution statistics

---

## Teams

Displays:

* Country
* FIFA Code
* Group
* Pool
* API aliases

Used primarily for transparency and debugging.

---

## Matches

Displays:

* Fixtures
* Scores
* Match status
* Feed information

Used for transparency and validation.

---

# Architecture Principles

## Derived State

Standings must never be manually edited.

Everything derives from matches.

```text
Matches
    ↓
Countries
    ↓
Participants
    ↓
Standings
```

---

## Separation of Concerns

World Cup Feed:

* Owns raw data.

Country Registry:

* Owns team identities.

Match Engine:

* Owns match normalisation.

Scoring Engine:

* Owns point calculations.

UI:

* Owns presentation only.

---

# Future Enhancements

## Live Updates

Current:

* Poll every 5 minutes.

Future:

* WebSockets.
* Server Sent Events.

---

## Team Ownership View

Selecting a country should show:

* Owners
* Points contributed
* Goals scored
* Upcoming matches

---

## Historical Movement

Track position changes.

Example:

```text
Yesterday #5
Today #2

↑ +3
```

---

## Predictions

Allow participants to predict:

* Match scores
* Tournament winner
* Golden boot

---

# Technical Preferences

Frontend:

* HTML
* Tailwind
* Vanilla JavaScript

Future:

* Vue 3
* TypeScript

Hosting:

* Cloudflare Pages

Data Source:

* World Cup API provider
* Adapter layer required

Persistence:

* Local storage initially
* Server-side persistence optional later

---

# Success Criteria

A participant can:

1. Open the application.
2. See the current sweepstake rankings.
3. See which teams they own.
4. Understand how points were earned.
5. Trust that rankings reflect real World Cup results.
