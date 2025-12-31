# CLAUDE.md

Guidance for Claude Code when working with this Twenty CRM fork.

## Simplicity Principle

Before making changes that add complexity, **ask the user first**:

- Adding new dependencies or packages
- Creating new abstractions (hooks, utilities, components)
- Changes spanning 3+ files
- Architectural modifications

Ask: "I'm planning to [change]. This involves [complexity]. Should I proceed?"

---

## Documentation

Detailed documentation is in `.agent_docs/`:

| Document | Purpose |
|----------|---------|
| [architecture.md](.agent_docs/architecture.md) | System design, deployment, tech stack |
| [datamodel.md](.agent_docs/datamodel.md) | Entity reference, relationships |
| [coding-standards.md](.agent_docs/coding-standards.md) | All coding conventions |
| [twentypattern.md](.agent_docs/twentypattern.md) | Twenty workspace entity patterns |
| [roadmap.md](.agent_docs/roadmap.md) | Development phases, tasks |
| [dev_log.md](.agent_docs/dev_log.md) | Development journal |

---

## Key Commands

### Development
```bash
twenty-dev            # Start DEV MODE (PM2) - fast iteration
twenty-docker         # Start DOCKER MODE - production testing
twenty-status         # Check current mode
npx nx start          # Start all services (alternative)
```

### Testing
```bash
# Single file (PREFERRED - fast)
npx jest path/to/test.test.ts --config=packages/twenty-front/jest.config.mjs
npx jest path/to/test.spec.ts --config=packages/twenty-server/jest.config.mjs

# Full project (slow)
npx nx test twenty-front
```

### Code Quality
```bash
npx nx lint:changed twenty-front              # Lint changed files (fast)
npx nx lint:changed twenty-front --configuration=fix
npx nx typecheck twenty-front
```

### Database
```bash
npx nx database:reset twenty-server
npx nx run twenty-server:typeorm migration:generate src/database/typeorm/core/migrations/common/[name] -d src/database/typeorm/core/core.datasource.ts
npx nx run twenty-server:command workspace:sync-metadata
```

---

## Project Structure

```
packages/
├── twenty-front/    # React app (port 3001)
├── twenty-server/   # NestJS API (port 3000)
├── twenty-ui/       # Shared components
└── twenty-shared/   # Common types
```

---

## OCI Deployment

### Mode Commands
```bash
twenty-dev            # DEV MODE - fast iteration, auto-reload backend
twenty-docker         # DOCKER MODE - production testing
twenty-status         # Check current mode
twenty-stop           # Stop all services
twenty-rebuild-front  # Rebuild frontend (DEV MODE only)
```

### Infrastructure
- PostgreSQL: localhost:5432
- Redis: localhost:6379
- Frontend: Port 3001
- Backend: Port 3000 (DEV) / 3002 (DOCKER)

---

## Database Safety

### Backup Commands
```bash
twenty-backup       # Create backup (keeps last 7)
twenty-restore      # Restore from backup
twenty-backup-list  # List available backups
```

Backups: `/home/ubuntu/workspace/twentyone/backups/`
Auto-backup: Daily at 3 AM

### DANGER - Data Loss Commands
```bash
docker compose down -v     # Deletes all data volumes
npx nx database:reset      # Wipes and reseeds database
```

### Safe Commands
```bash
twenty-dev          # Keeps database running
twenty-stop         # Stops apps, keeps database
docker compose down # Data preserved in volume
```

---

## Skills

Use `/orchestrator`, `/architect`, `/frontend-dev`, `/backend-dev`, `/qa`, `/devops`

See `.claude/skills/WORKFLOW.md` for details.
