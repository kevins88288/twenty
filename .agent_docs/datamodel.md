# High Signal VC CRM - Data Model

## Implementation Note

All entities below are implemented as **Twenty Workspace Entities** (standard objects defined in code), except for `transcript_segments` which is a separate PostgreSQL table.

See `agent_docs/TWENTY_PATTERNS.md` for how to create these as workspace entities.

## Entity Relationship Overview

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│      Firm       │────<│  FirmEngagement  │>────│ FundraiseRound  │
│ (firmEmail)     │     │   (junction)     │     │                 │
└────────┬────────┘     └────────┬─────────┘     └─────────────────┘
         │                       │
         │                       │
    ┌────┴────┐            ┌─────┴─────┐
    │         │            │           │
┌───┴───┐  ┌──┴──┐   ┌─────┴────┐ ┌────┴─────┐
│Person │  │Notes│   │Interaction│ │PitchMeeting│
└───────┘  └─────┘   └──────────┘ └─────┬─────┘
                                        │
                                 ┌──────┴──────┐
                                 │TranscriptSeg│
                                 │  (separate  │
                                 │   table)    │
                                 └─────────────┘
```

## Key Design Decisions

1. **Firm keyed by email domain** (`firmEmail`), not name - handles "Sequoia" ambiguity
2. **FirmEngagement is the junction** - enables per-round status tracking
3. **Interaction links to FirmEngagement** - every interaction has round context
4. **last_interaction_date is computed** - via MAX(Interaction.date) JOIN, not stored
5. **"Default" round for pre-raise** - captures inbound before formal fundraise starts

---

## Workspace Entity: Firm

**Location**: `packages/twenty-server/src/modules/firm/standard-objects/firm.workspace-entity.ts`

VC firm identified by email domain. Primary entity for organizing investor relationships.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| **firmEmail** | TEXT | ✅ | Unique identifier - Email domain (e.g., sequoiacap.com) |
| name | TEXT | ✅ | Display name (e.g., Sequoia Capital) |
| website | LINK | | Firm website |
| stageFocus | MULTI_SELECT | | Pre-seed, Seed, Series A, Series B, Growth |
| sectorFocus | MULTI_SELECT | | Fintech, Crypto, Enterprise, Consumer, Healthcare, etc. |
| geoFocus | MULTI_SELECT | | US, Canada, LATAM, MENA, APAC, Europe, etc. |
| fundSize | CURRENCY | | Current fund size in USD |
| typicalCheckMin | CURRENCY | | Minimum check size |
| typicalCheckMax | CURRENCY | | Maximum check size |
| notes | RICH_TEXT | | General notes about the firm |

**Relationships:**
- Has many → Person (employees)
- Has many → FirmEngagement (one per fundraise round)

---

## Workspace Entity: Person (Extended)

**Location**: `packages/twenty-server/src/modules/person/standard-objects/person.workspace-entity.ts`

We extend Twenty's built-in Person object with new fields for VC tracking.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| (inherited from Twenty) | | | firstName, lastName, email, phone, etc. |
| currentFirm | RELATION | | Link to Firm (MANY_TO_ONE) |
| role | SELECT | | Partner, Principal, Associate, Analyst, Venture Partner |
| linkedinUrl | LINK | | LinkedIn profile |
| twitterHandle | TEXT | | Twitter/X handle |
| telegramHandle | TEXT | | Telegram username |
| introSource | TEXT | | How connected (warm intro from X, cold, conference, etc.) |

**Relationships:**
- Belongs to → Firm (current employer)
- Has many → Interaction
- Has many → PitchMeeting (as participant)

**Note:** When a partner moves firms, update `currentFirm` but interaction history stays linked to the Person, not the firm.

---

## Workspace Entity: FundraiseRound

**Location**: `packages/twenty-server/src/modules/fundraise-round/standard-objects/fundraise-round.workspace-entity.ts`

Represents a fundraising round. Create a "Default" round for inbound before formal raise.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | TEXT | ✅ | Round name (e.g., "Default", "Pre-seed 2024", "Seed 2025") |
| roundType | SELECT | ✅ | Default, Pre-seed, Seed, Series A, Series B, Bridge |
| targetAmount | CURRENCY | | Target raise amount |
| status | SELECT | | Active, Paused, Closed - Successful, Closed - Unsuccessful |
| startDate | DATE | | When fundraising started |
| closeDate | DATE | | When round closed (if applicable) |
| isCurrent | BOOLEAN | | Quick filter for active round (only one should be true) |

**Relationships:**
- Has many → FirmEngagement

---

## Workspace Entity: FirmEngagement

**Location**: `packages/twenty-server/src/modules/firm-engagement/standard-objects/firm-engagement.workspace-entity.ts`

Junction between Firm and FundraiseRound. **This is the core entity for pipeline tracking.**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| firm | RELATION | ✅ | Link to Firm (MANY_TO_ONE) |
| fundraiseRound | RELATION | ✅ | Link to FundraiseRound (MANY_TO_ONE) |
| status | SELECT | ✅ | 21 status options (see below) |
| priority | SELECT | | High, Medium, Low |
| primaryContact | RELATION | | Link to Person (MANY_TO_ONE) |
| notes | RICH_TEXT | | Round-specific notes (rejection reasons, etc.) |

**Computed Fields (via query, not stored):**
- `last_interaction_date`: MAX(Interaction.date) WHERE firm_engagement_id = this.id

**Relationships:**
- Belongs to → Firm
- Belongs to → FundraiseRound
- Has many → Interaction
- Has many → PitchMeeting

**Unique Constraint:** (firm_id, fundraise_round_id) - one engagement per firm per round

### Status Values (21 options)

**Pipeline Stages (11):**
1. Prospecting
2. Warm Intro Needed
3. Warm Intro Sent
4. Cold Outreach Sent
5. 1st Meeting Scheduled
6. 1st Meeting Completed
7. Partner Meeting
8. DD (Due Diligence)
9. Investment Committee
10. Term Sheet
11. Committed

**Special States (3):**
- Went Cold - Need Follow Up
- Want Lead (they want to lead the round)
- On Ice (parked for later)

**Rejection Reasons (7):**
- Rejected - Generic
- Rejected - Too High Val
- Rejected - Not Fit Geo
- Rejected - Thesis
- Rejected - Competitor
- Rejected - Post DD/IC
- Rejected - Post Traction
- Avoid (blacklist)

---

## Workspace Entity: Interaction

**Location**: `packages/twenty-server/src/modules/interaction/standard-objects/interaction.workspace-entity.ts`

General interactions with a Person. Links to FirmEngagement for round context.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| person | RELATION | ✅ | Link to Person (MANY_TO_ONE) |
| firmEngagement | RELATION | ✅ | Link to FirmEngagement (MANY_TO_ONE) |
| channel | SELECT | ✅ | Email, LinkedIn, Telegram, WhatsApp, Twitter, Phone, Other |
| direction | SELECT | ✅ | Inbound, Outbound |
| date | DATE_TIME | ✅ | When interaction occurred |
| subject | TEXT | | Brief description or email subject |
| content | RICH_TEXT | | Full content or notes |
| outcome | SELECT | | No Response, Positive, Neutral, Negative, Meeting Booked |

**Relationships:**
- Belongs to → Person
- Belongs to → FirmEngagement

**Note:** Emails auto-populate from Gmail sync. Other channels logged manually.

---

## Workspace Entity: PitchMeeting

**Location**: `packages/twenty-server/src/modules/pitch-meeting/standard-objects/pitch-meeting.workspace-entity.ts`

Pitch calls with transcript storage. Links to FirmEngagement for round context.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| title | TEXT | ✅ | Meeting title (from Granola or manual) |
| date | DATE_TIME | ✅ | When pitch occurred |
| firmEngagement | RELATION | ✅ | Link to FirmEngagement (MANY_TO_ONE) |
| participants | RELATION | | Links to People (MANY_TO_MANY) |
| meetingType | SELECT | | First Call, Partner Meeting, DD Call, IC Presentation |
| rawTranscript | RICH_TEXT | | Full Granola transcript |
| aiSummary | RICH_TEXT | | AI-generated summary |
| processingStatus | SELECT | | Pending, Processing, Completed, Failed |

**Relationships:**
- Belongs to → FirmEngagement
- Has many → Person (participants)
- Has many → TranscriptSegment (separate table)

---

## PostgreSQL Table: transcript_segments

**NOT a workspace entity** - this is a separate PostgreSQL table created via TypeORM migration.

**Location**: `packages/twenty-server/src/database/typeorm/core/migrations/`

Parsed segments from transcripts. Separate for performance and specialized indexing.

| Column | Type | Description |
|--------|------|-------------|
| id | UUID | Primary key |
| pitch_meeting_id | UUID | FK to PitchMeeting |
| sequence_order | INTEGER | Order within transcript |
| content | TEXT | Segment text |
| speaker_role | TEXT | 'founder' or 'vc' |
| segment_type | TEXT | 'question', 'answer', 'pitch', 'objection', 'smalltalk' |
| topic_tags | TEXT[] | Array of topics (competition, unit_economics, team, market, etc.) |
| linked_segment_id | UUID | FK to linked Q or A (for Q&A pairs) |
| search_vector | TSVECTOR | Generated for full-text search |

**Indexes:**
- GIN on topic_tags
- GIN on search_vector
- B-tree on pitch_meeting_id

---

## Example Queries

### Get firm's status in current round
```sql
SELECT f.name, fe.status, fe.priority
FROM firm f
JOIN firm_engagement fe ON f.id = fe.firm_id
JOIN fundraise_round fr ON fe.fundraise_round_id = fr.id
WHERE fr.is_current = true
ORDER BY fe.status;
```

### Get last interaction date per engagement
```sql
SELECT fe.id, MAX(i.date) as last_interaction_date
FROM firm_engagement fe
LEFT JOIN interaction i ON fe.id = i.firm_engagement_id
GROUP BY fe.id;
```

### Find all VC questions about competition
```sql
SELECT ts.content, pm.title, pm.date
FROM transcript_segments ts
JOIN pitch_meeting pm ON ts.pitch_meeting_id = pm.id
WHERE ts.speaker_role = 'vc'
  AND ts.segment_type = 'question'
  AND 'competition' = ANY(ts.topic_tags)
ORDER BY pm.date DESC;
```

### Find stale engagements (no interaction in 14+ days)
```sql
SELECT f.name, fe.status, MAX(i.date) as last_contact
FROM firm_engagement fe
JOIN firm f ON fe.firm_id = f.id
JOIN fundraise_round fr ON fe.fundraise_round_id = fr.id
LEFT JOIN interaction i ON fe.id = i.firm_engagement_id
WHERE fr.is_current = true
  AND fe.status NOT IN ('Committed', 'Rejected - Generic', ...) -- non-terminal
GROUP BY f.name, fe.status
HAVING MAX(i.date) < NOW() - INTERVAL '14 days'
   OR MAX(i.date) IS NULL;
```