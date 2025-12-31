# High Signal VC CRM - Architecture

## Tech Stack Summary

- **Frontend**: React 18, TypeScript, Recoil, Emotion, Vite
- **Backend**: NestJS, TypeORM, PostgreSQL, Redis, GraphQL Yoga
- **Monorepo**: Nx workspace, Yarn 4
- **AI**: Claude API for transcript processing

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Docker Compose                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐ │
│  │ Twenty Front │   │ Twenty Server│   │   AI Processing      │ │
│  │   (React)    │──>│   (NestJS)   │──>│   Service (Python)   │ │
│  │   Port 3001  │   │   Port 3000  │   │                      │ │
│  └──────────────┘   └──────┬───────┘   └──────────┬───────────┘ │
│                            │                       │             │
│                    ┌───────┴───────┐              │             │
│                    │               │              │             │
│              ┌─────┴─────┐   ┌─────┴─────┐  ┌────┴────┐        │
│              │PostgreSQL │   │  Redis    │  │ Claude  │        │
│              │  Port 5432│   │ Port 6379 │  │   API   │        │
│              └───────────┘   └───────────┘  └─────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

## Twenty CRM Architecture (Base Platform)

### Frontend (packages/twenty-front)

- **Framework**: React 18 with TypeScript
- **State Management**: Recoil (atoms/selectors pattern)
- **Data Fetching**: Apollo Client (GraphQL)
- **UI Components**: Custom design system in twenty-ui package
- **Routing**: React Router

Key directories:
```
twenty-front/src/
├── modules/                 # Feature modules
│   ├── object-record/       # Generic record CRUD
│   ├── ui/                  # UI components
│   ├── activities/          # Notes, tasks, etc.
│   └── settings/            # Workspace settings
├── pages/                   # Route pages
├── generated/               # Apollo codegen output
└── utils/                   # Utilities
```

### Backend (packages/twenty-server)

- **Framework**: NestJS with TypeScript
- **API**: GraphQL (primary) + REST (secondary)
- **ORM**: TypeORM with PostgreSQL
- **Auth**: JWT + Google OAuth
- **Jobs**: BullMQ with Redis
- **Email Sync**: Google API, Microsoft Graph, IMAP

Key directories:
```
twenty-server/src/
├── engine/                  # Core engine
│   ├── api/                 # GraphQL/REST API
│   ├── core-modules/        # Auth, file, billing
│   └── metadata-modules/    # Object/field definitions
├── modules/                 # Business modules
│   ├── messaging/           # Email sync
│   ├── calendar/            # Calendar sync
│   └── workflow/            # Workflow automation
└── database/                # Migrations, seeds
```

### Standard Objects (Workspace Entities)

Twenty uses a **metadata-driven** approach. Standard objects are defined as TypeScript classes with decorators:

1. **Workspace Entity**: Class decorated with `@WorkspaceEntity` defines an object
2. **Workspace Fields**: Properties decorated with `@WorkspaceField` define fields
3. **Workspace Relations**: Properties decorated with `@WorkspaceRelation` define relationships
4. **Sync Metadata**: Running `workspace:sync-metadata` applies definitions to database and generates GraphQL

Standard objects are defined in `packages/twenty-server/src/modules/`:

```
modules/
├── person/
│   ├── standard-objects/
│   │   └── person.workspace-entity.ts   # Person definition
│   └── person.module.ts
├── company/
│   └── standard-objects/
│       └── company.workspace-entity.ts  # Company definition
└── opportunity/
    └── standard-objects/
        └── opportunity.workspace-entity.ts
```

**How to add new objects**: Create a new workspace entity file following the same pattern. See `agent_docs/TWENTY_PATTERNS.md` for full details.

---

## Our Customizations Architecture

### Layer 1: Standard Objects (Workspace Entities in Code)

We add these as new standard objects in `packages/twenty-server/src/modules/`:

| Object | Location | Notes |
|--------|----------|-------|
| Firm | `modules/firm/` | firmEmail as unique identifier |
| FundraiseRound | `modules/fundraise-round/` | Tracks each raise |
| FirmEngagement | `modules/firm-engagement/` | Junction: Firm per Round with status |
| Interaction | `modules/interaction/` | Links to FirmEngagement |
| PitchMeeting | `modules/pitch-meeting/` | Transcript metadata |

We also **extend** the existing Person entity with new fields (role, currentFirm, telegramHandle).

### Layer 2: Separate PostgreSQL Tables

These tables are **outside** Twenty's ORM for performance:

| Table | Purpose |
|-------|---------|
| transcript_segments | Parsed transcript chunks with search vectors |

Why separate? 
- Write-once-read-many pattern (doesn't fit Twenty's CRUD model)
- Need specialized indexes (GIN, tsvector)
- Future migration to Qdrant

### Layer 3: AI Processing Service

Separate Python service for transcript processing:

```
┌─────────────────────────────────────────────────────────┐
│                AI Processing Service                     │
├─────────────────────────────────────────────────────────┤
│  Webhook Receiver                                        │
│       │                                                  │
│       ▼                                                  │
│  Transcript Parser (Me:/Them: → founder/vc)             │
│       │                                                  │
│       ▼                                                  │
│  Segment Classifier (question/answer/pitch/objection)   │
│       │                                                  │
│       ▼                                                  │
│  Topic Extractor (competition, unit_economics, etc.)    │
│       │                                                  │
│       ▼                                                  │
│  Q&A Linker (match questions to answers)                │
│       │                                                  │
│       ▼                                                  │
│  Summary Generator                                       │
│       │                                                  │
│       ▼                                                  │
│  PostgreSQL Writer (transcript_segments table)          │
│       │                                                  │
│       ▼                                                  │
│  Twenty API Callback (update PitchMeeting.ai_summary)   │
└─────────────────────────────────────────────────────────┘
```

---

## Integration Points

### 1. Gmail Sync (Built-in Twenty)

Twenty's messaging module syncs emails automatically. We extend it:

```
Email received → Twenty syncs to Person
                      │
                      ▼ (Workflow trigger)
            Find Person.current_firm
                      │
                      ▼
            Find active FirmEngagement
                      │
                      ▼
            Create Interaction record
```

**Configuration**: Settings > Accounts > Add Gmail account

### 2. Workflow Automation (Built-in Twenty)

Twenty's workflow engine handles automations:

| Workflow | Trigger | Action |
|----------|---------|--------|
| Auto-status on meeting | PitchMeeting created | Update FirmEngagement status |
| Stale alert | Weekly schedule | Create Tasks for stale engagements |
| Email to Interaction | Email synced | Create Interaction linked to engagement |

**Configuration**: Settings > Workflows

### 3. Webhooks for Transcript Processing

Twenty fires webhooks on record changes:

```
PitchMeeting updated (raw_transcript field)
           │
           ▼
    Webhook POST to AI service
           │
           ▼
    AI processes transcript
           │
           ▼
    AI calls Twenty API to update ai_summary
```

**Configuration**: Settings > API & Webhooks

---

## Data Flow Examples

### Creating a New Fundraise Round

```
1. User creates FundraiseRound (name: "Seed 2025", is_current: true)
2. User bulk-creates FirmEngagements for target firms
   - Each starts with status: "Prospecting"
   - Links to new round
3. Previous round's is_current set to false
```

### Logging a Pitch Meeting

```
1. User creates PitchMeeting
   - Links to FirmEngagement
   - Pastes raw_transcript from Granola
   - Sets processing_status: "Pending"

2. Webhook fires to AI service

3. AI service:
   - Parses transcript into segments
   - Classifies each segment
   - Extracts topics
   - Links Q&A pairs
   - Generates summary
   - Writes to transcript_segments table
   - Calls Twenty API to update:
     - PitchMeeting.ai_summary
     - PitchMeeting.processing_status = "Completed"

4. Workflow triggers:
   - If FirmEngagement.status = "1st Meeting Scheduled"
   - Update to "1st Meeting Completed"
   - Create follow-up Task
```

### Searching Transcripts

**MVP (PostgreSQL full-text):**
```sql
SELECT * FROM transcript_segments
WHERE search_vector @@ to_tsquery('competition & market')
  AND speaker_role = 'vc'
  AND segment_type = 'question';
```

**Future (Qdrant semantic):**
```python
client.search(
    collection_name='transcript_segments',
    query_vector=embed('how do you think about competition'),
    query_filter={
        'must': [
            {'key': 'speaker_role', 'match': {'value': 'vc'}},
            {'key': 'segment_type', 'match': {'value': 'question'}}
        ]
    }
)
```

---

## Deployment Architecture

### Local Development

```yaml
# docker-compose.yml
services:
  twenty-front:
    build: ./packages/twenty-front
    ports: ["3001:3001"]
    
  twenty-server:
    build: ./packages/twenty-server
    ports: ["3000:3000"]
    environment:
      - DATABASE_URL=postgres://...
      
  postgres:
    image: postgres:15
    ports: ["5432:5432"]
    volumes:
      - postgres_data:/var/lib/postgresql/data
      
  redis:
    image: redis:7
    ports: ["6379:6379"]
    
  ai-service:
    build: ./ai-service
    ports: ["8000:8000"]
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
```

### Future: Qdrant Addition

```yaml
  qdrant:
    image: qdrant/qdrant
    ports: ["6333:6333"]
    volumes:
      - qdrant_data:/qdrant/storage
```

---

## Security Considerations

1. **API Keys**: Stored in environment variables, never committed
2. **Transcript Data**: Stored locally, processed via user's own Claude API key
3. **Gmail OAuth**: Standard OAuth 2.0 flow, tokens encrypted at rest
4. **Database**: Local PostgreSQL, optional encryption at rest

---

## Performance Considerations

1. **Pipeline View**: Index on (fundraise_round_id, status) for fast filtering
2. **Transcript Search**: GIN index on tsvector, future HNSW index in Qdrant
3. **Last Interaction**: Computed via JOIN, consider materialized view if slow
4. **Email Sync**: Background job, doesn't block UI

---

## OCI Deployment (Production Server)

This deployment uses a dual-mode setup for development vs production testing.

### Dual-Mode Setup

| Mode | Purpose | Startup |
|------|---------|---------|
| **DEV MODE (PM2)** | Fast iteration, auto-reload backend | `twenty-dev` (~5 sec) |
| **DOCKER MODE** | Production testing with local builds | `twenty-docker` (~2-4 min) |

### Mode Commands

```bash
twenty-dev            # Start DEV MODE - fast iteration
twenty-docker         # Start DOCKER MODE - production testing
twenty-status         # Check current mode
twenty-stop           # Stop all services
twenty-logs           # View logs for current mode
twenty-rebuild-front  # Rebuild frontend (DEV MODE only)
```

### Infrastructure

PostgreSQL and Redis always run in Docker containers (shared by both modes):

| Service | Port | Notes |
|---------|------|-------|
| PostgreSQL | 5432 | Database container |
| Redis | 6379 | Cache/queue container |
| Frontend | 3001 | React app |
| Backend | 3000 (DEV) / 3002 (DOCKER) | NestJS API |

### Database Safety

**Backup Commands:**
```bash
twenty-backup       # Create backup (keeps last 7)
twenty-restore      # Restore from backup
twenty-backup-list  # List available backups
```

Backups stored in: `/home/ubuntu/workspace/twentyone/backups/`
Automatic daily backup at 3 AM.

**DANGER - Commands That DELETE Data:**
```bash
docker compose down -v          # Deletes all data volumes
docker volume rm <volume>       # Deletes specific volume
npx nx database:reset           # Wipes and reseeds database
```

### Configuration Files

- `ecosystem.config.js` - PM2 process configuration
- `packages/twenty-docker/docker-compose.local.yml` - Local Docker build
- `scripts/twenty-*.sh` - Mode switching scripts