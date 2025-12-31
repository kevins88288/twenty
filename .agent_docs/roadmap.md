# High Signal VC CRM - Feature Roadmap

## Quick Reference

Use task codes (e.g., "Plan 1C", "Execute 2A") to request specific work.

| Code | Task | Dependencies |
|------|------|--------------|
| **1A** | Environment Setup | - |
| **1B** | Study Workspace Patterns | 1A |
| **1C** | Create Firm Entity | 1B |
| **1D** | Extend Person with VC Fields | 1C |
| **1E** | Create FundraiseRound Entity | 1B |
| **1F** | Configure Gmail Sync | 1A |
| **2A** | Create FirmEngagement Entity | 1C, 1E |
| **2B** | Create Interaction Entity | 1D, 2A |
| **2C** | Create Pipeline Views (UI) | 2A |
| **2D** | Configure Workflows (UI) | 2A, 2B, 3A |
| **3A** | Create PitchMeeting Entity | 2A |
| **3B** | Create transcript_segments Table | 3A |
| **3C** | Build AI Processing Service | 3A, 3B |
| **3D** | Configure Transcript Webhook | 3C |
| **4A** | Full-Text Search UI | 3C |
| **4B** | Import Existing VC Data | 1C, 1D, 1E |
| **4C** | Dashboard Setup | 2A |
| **4D** | Documentation & README | - |

---

## Development Workflow

1. **Read TWENTY_PATTERNS.md first** - Understand workspace entity patterns before any coding
2. **Study existing objects** - Look at person/, company/, opportunity/ modules as reference
3. **One task per session** - Clear context between distinct tasks
4. **Run sync-metadata** - After any entity changes: `yarn command:dev workspace:sync-metadata -f`
5. **Verify in UI** - Check Settings > Data Model after syncing

## Current Status

- [x] PRD defined
- [x] Data model designed
- [x] Architecture documented
- [ ] **Phase 1: Foundation** ← CURRENT
- [ ] Phase 2: Engagements & Interactions
- [ ] Phase 3: Transcripts
- [ ] Phase 4: Search & Polish

---

## Phase 1: Foundation

**Goal**: Fork Twenty, set up local dev, create base standard objects in code

### 1A: Environment Setup
**Status**: Not started | **Priority**: P0

**Scope**: Fork, clone, configure Twenty CRM locally

**Steps**:
1. Fork Twenty repository to your GitHub
2. Clone locally
3. Copy `.env.example` to `.env` and configure
4. Run `yarn install`
5. Run `docker-compose up` for PostgreSQL and Redis
6. Run `yarn dev` to start frontend and backend
7. Create initial workspace via UI

**Verification**:
- [ ] Can access Twenty UI at localhost:3001
- [ ] Can log in and see default objects (People, Companies)
- [ ] GraphQL playground works at localhost:3000/graphql

**Files to read first**:
- Twenty's README.md
- Twenty's docker-compose.yml
- Twenty's .env.example

---

### 1B: Study Workspace Entity Patterns
**Status**: Not started | **Priority**: P0

**Scope**: Research only - understand Twenty's entity system (no code changes)

**Steps**:
1. Read `agent_docs/TWENTY_PATTERNS.md` completely
2. Explore `packages/twenty-server/src/modules/person/`
   - Study `standard-objects/person.workspace-entity.ts`
   - Note decorators: @WorkspaceEntity, @WorkspaceField, @WorkspaceRelation
   - Note how standard field IDs are defined
3. Explore `packages/twenty-server/src/modules/company/`
4. Explore `packages/twenty-server/src/modules/opportunity/`
5. Find where these modules are registered (likely in a parent module)
6. Document any patterns not covered in TWENTY_PATTERNS.md

**Verification**:
- [ ] Understand @WorkspaceEntity decorator parameters
- [ ] Understand @WorkspaceField types and options
- [ ] Understand @WorkspaceRelation for one-to-many and many-to-one
- [ ] Know where to register new modules

---

### 1C: Create Firm Workspace Entity
**Status**: Not started | **Priority**: P0 | **Depends on**: 1B

**Scope**: New workspace entity for VC firms

**Steps**:
1. Create directory: `packages/twenty-server/src/modules/firm/`
2. Create `standard-objects/firm.workspace-entity.ts` following Person pattern
3. Create `standard-objects/firm.standard-field-ids.ts` with unique UUIDs
4. Create `firm.module.ts`
5. Register FirmModule in parent module (where PersonModule is registered)
6. Run `yarn command:dev workspace:sync-metadata -f`

**Fields to create**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| firmEmail | TEXT | Yes | - |
| name | TEXT | Yes | - |
| website | LINK | No | - |
| stageFocus | MULTI_SELECT | No | Pre-seed, Seed, Series A, Series B, Growth |
| sectorFocus | MULTI_SELECT | No | Fintech, Crypto, Enterprise, Consumer, Healthcare |
| geoFocus | MULTI_SELECT | No | US, Canada, LATAM, MENA, APAC, Europe |
| fundSize | CURRENCY | No | - |
| typicalCheckMin | CURRENCY | No | - |
| typicalCheckMax | CURRENCY | No | - |
| notes | RICH_TEXT | No | - |

**Verification**:
- [ ] Firm appears in Settings > Data Model
- [ ] Can create a Firm record via UI
- [ ] All fields render correctly
- [ ] GraphQL queries work: `query { firms { edges { node { id name firmEmail } } } }`

---

### 1D: Extend Person with VC Fields
**Status**: Not started | **Priority**: P0 | **Depends on**: 1C

**Scope**: Add investor-specific fields to existing Person entity

**Steps**:
1. Open `packages/twenty-server/src/modules/person/standard-objects/person.workspace-entity.ts`
2. Add new fields to PersonWorkspaceEntity class
3. Add new field IDs to person.standard-field-ids.ts
4. Run `yarn command:dev workspace:sync-metadata -f`

**Fields to add**:
| Field | Type | Options |
|-------|------|---------|
| currentFirm | RELATION (→Firm) | MANY_TO_ONE, nullable |
| role | SELECT | Partner, Principal, Associate, Analyst, Venture Partner |
| linkedinUrl | LINK | nullable |
| twitterHandle | TEXT | nullable |
| telegramHandle | TEXT | nullable |
| introSource | TEXT | nullable |

**Verification**:
- [ ] New fields appear on Person in UI
- [ ] Can link Person to Firm via currentFirm relation
- [ ] Role dropdown works
- [ ] Firm.people relation shows linked people

---

### 1E: Create FundraiseRound Workspace Entity
**Status**: Not started | **Priority**: P0 | **Depends on**: 1B

**Scope**: Entity to track funding rounds

**Steps**:
1. Create directory: `packages/twenty-server/src/modules/fundraise-round/`
2. Create workspace entity with fields
3. Create standard field IDs
4. Create module and register
5. Run sync-metadata

**Fields**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| name | TEXT | Yes | e.g., "Seed 2025", "Default" |
| roundType | SELECT | No | Default, Pre-seed, Seed, Series A, Series B, Bridge |
| targetAmount | CURRENCY | No | - |
| status | SELECT | No | Active, Paused, Closed - Successful, Closed - Unsuccessful |
| startDate | DATE | No | - |
| closeDate | DATE | No | - |
| isCurrent | BOOLEAN | No | default: false |

**Verification**:
- [ ] FundraiseRound appears in Data Model
- [ ] Can create a "Default" round
- [ ] isCurrent toggle works

---

### 1F: Configure Gmail Sync
**Status**: Not started | **Priority**: P1 | **Depends on**: 1A

**Scope**: Enable email synchronization

**Steps**:
1. Set up Google Cloud project with OAuth credentials
2. Add to .env:
   ```
   MESSAGING_PROVIDER_GMAIL_ENABLED=true
   AUTH_GOOGLE_CLIENT_ID=xxx
   AUTH_GOOGLE_CLIENT_SECRET=xxx
   ```
3. Restart server
4. Connect Gmail in Settings > Accounts
5. Test email sync

**Verification**:
- [ ] Gmail account connected
- [ ] Emails sync to matching Person records
- [ ] Can view email content in Person detail view

---

## Phase 2: Engagements & Interactions

**Goal**: Create FirmEngagement (junction with 21 statuses), Interaction entity, workflows

### 2A: Create FirmEngagement Workspace Entity
**Status**: Not started | **Priority**: P0 | **Depends on**: 1C, 1E

**Scope**: Junction entity linking Firms to FundraiseRounds with 21-status pipeline

**Steps**:
1. Create `packages/twenty-server/src/modules/firm-engagement/`
2. Create workspace entity with 21-status SELECT field
3. Create relations to Firm and FundraiseRound
4. Run sync-metadata

**Fields**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| firm | RELATION (→Firm) | Yes | MANY_TO_ONE |
| fundraiseRound | RELATION (→FundraiseRound) | Yes | MANY_TO_ONE |
| status | SELECT | No | 21 options (see TWENTY_PATTERNS.md) |
| priority | SELECT | No | High, Medium, Low |
| primaryContact | RELATION (→Person) | No | MANY_TO_ONE |
| notes | RICH_TEXT | No | - |

**Verification**:
- [ ] Can create FirmEngagement linking Firm to Round
- [ ] All 21 status options visible in dropdown
- [ ] Kanban view groups by status

---

### 2B: Create Interaction Workspace Entity
**Status**: Not started | **Priority**: P0 | **Depends on**: 1D, 2A

**Scope**: Log individual touchpoints with investors

**Steps**:
1. Create `packages/twenty-server/src/modules/interaction/`
2. Create workspace entity
3. Run sync-metadata

**Fields**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| person | RELATION (→Person) | Yes | MANY_TO_ONE |
| firmEngagement | RELATION (→FirmEngagement) | Yes | MANY_TO_ONE |
| channel | SELECT | No | Email, LinkedIn, Telegram, WhatsApp, Twitter, Phone, Other |
| direction | SELECT | No | Inbound, Outbound |
| date | DATE_TIME | Yes | - |
| subject | TEXT | No | - |
| content | RICH_TEXT | No | - |
| outcome | SELECT | No | No Response, Positive, Neutral, Negative, Meeting Booked |

**Verification**:
- [ ] Can create Interaction linked to Person and FirmEngagement
- [ ] Interactions visible on FirmEngagement detail page

---

### 2C: Create Pipeline Views (UI Configuration)
**Status**: Not started | **Priority**: P0 | **Depends on**: 2A

**Scope**: UI configuration - create filtered views (no code changes)

**Steps** (in UI after objects exist):
1. Go to FirmEngagement table view
2. Create view: "Active Pipeline" - filter isCurrent=true, Kanban by status
3. Create view: "Need Follow-up" - status = WENT_COLD
4. Create view: "Hot Leads" - status in (PARTNER_MEETING, DD, INVESTMENT_COMMITTEE, TERM_SHEET)
5. Create view: "All Rejections" - status starts with REJECTED

**Verification**:
- [ ] Kanban shows firms grouped by status
- [ ] Can drag firms between status columns

---

### 2D: Configure Workflows (UI Configuration)
**Status**: Not started | **Priority**: P1 | **Depends on**: 2A, 2B, 3A

**Scope**: Automation rules in Settings > Workflows (no code changes)

**Workflows to create**:

1. **Auto-status on Meeting**
   - Trigger: PitchMeeting created
   - Condition: FirmEngagement.status = FIRST_MEETING_SCHEDULED
   - Action: Update status to FIRST_MEETING_COMPLETED
   - Action: Create follow-up Task

2. **Stale Engagement Alert**
   - Trigger: Weekly schedule
   - Action: Find engagements with no interaction in 14 days
   - Action: Create Tasks

---

## Phase 3: Transcripts

**Goal**: PitchMeeting entity, transcript_segments table, AI processing

### 3A: Create PitchMeeting Workspace Entity
**Status**: Not started | **Priority**: P0 | **Depends on**: 2A

**Scope**: Entity for investor meetings with transcript support

**Fields**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| title | TEXT | Yes | - |
| date | DATE_TIME | Yes | - |
| firmEngagement | RELATION (→FirmEngagement) | Yes | MANY_TO_ONE |
| participants | RELATION (→Person) | No | MANY_TO_MANY |
| meetingType | SELECT | No | First Call, Partner Meeting, DD Call, IC Presentation |
| rawTranscript | RICH_TEXT | No | - |
| aiSummary | RICH_TEXT | No | - |
| processingStatus | SELECT | No | Pending, Processing, Completed, Failed |

---

### 3B: Create transcript_segments Table
**Status**: Not started | **Priority**: P0 | **Depends on**: 3A

**Scope**: Direct PostgreSQL table (NOT a Twenty workspace entity)

**Steps**:
1. Create TypeORM migration in `packages/twenty-server/src/database/typeorm/core/migrations/`
2. Create table with columns per DATA_MODEL.md
3. Add GIN indexes for topic_tags and search_vector
4. Run migration

**Reference**: See DATA_MODEL.md for column specifications

---

### 3C: Build AI Processing Service
**Status**: Not started | **Priority**: P0 | **Depends on**: 3A, 3B

**Scope**: Backend service for transcript analysis

**Reference**: See `TRANSCRIPT_PIPELINE.md` for full implementation details

---

### 3D: Configure Transcript Webhook
**Status**: Not started | **Priority**: P0 | **Depends on**: 3C

**Scope**: Connect transcript upload to AI processing

**Steps**:
1. Go to Settings > API & Webhooks
2. Create webhook for pitchMeeting.updated event
3. Point to AI processing service endpoint

---

## Phase 4: Search & Polish

### 4A: Full-Text Search UI
**Status**: Not started | **Priority**: P1 | **Depends on**: 3C

**Scope**: Search interface for transcript segments

---

### 4B: Import Existing VC Data
**Status**: Not started | **Priority**: P1 | **Depends on**: 1C, 1D, 1E

**Scope**: Bulk import of existing investor contacts/firms

---

### 4C: Dashboard Setup
**Status**: Not started | **Priority**: P2 | **Depends on**: 2A

**Scope**: Analytics dashboard (Labs feature)

---

### 4D: Documentation & README
**Status**: Not started | **Priority**: P2 | **Depends on**: -

**Scope**: Project documentation

---

## Future Phases (Not Yet Indexed)

### Phase 5: Qdrant Migration
- Deploy Qdrant container
- Generate embeddings for existing segments
- Build semantic search API
- Migrate queries from PostgreSQL

### Phase 6: Enhanced Integrations
- LinkedIn message logging (browser extension)
- Telegram sync
- Calendar integration for meeting detection

### Phase 7: Analytics
- Question frequency analysis
- Sentiment trends
- Conversion funnel metrics

---

## Dependency Graph

```
Phase 1:
1A ──► 1B ──► 1C ──► 1D
        │      │
        │      └──► 1E
        └──► 1F

Phase 2:
1C + 1E ──► 2A ──► 2B
             │      │
             └──► 2C │
                     └──► 2D (also needs 3A)

Phase 3:
2A ──► 3A ──► 3B ──► 3C ──► 3D

Phase 4:
3C ──► 4A
1C + 1D + 1E ──► 4B
2A ──► 4C
4D (independent)
```