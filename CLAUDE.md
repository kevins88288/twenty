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

### Docker Management
All commands run from: `cd ~/workspace/twentyone/packages/twenty-docker`

```bash
# Start/Stop
docker compose -f docker-compose.local.yml up -d       # Start all services
docker compose -f docker-compose.local.yml stop        # Stop all services
docker compose -f docker-compose.local.yml down        # Stop and remove containers

# Restart
docker compose -f docker-compose.local.yml restart server   # Restart server
docker compose -f docker-compose.local.yml restart worker   # Restart worker

# Rebuild (after code changes)
docker compose -f docker-compose.local.yml build            # Rebuild image
docker compose -f docker-compose.local.yml up -d            # Start with new image

# Logs
docker compose -f docker-compose.local.yml logs -f          # All logs
docker compose -f docker-compose.local.yml logs -f server   # Server logs

# Status
docker compose -f docker-compose.local.yml ps               # Container status
```

Shortcut alias available: `dc` (e.g., `dc up -d`, `dc logs -f`)

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

## Deployment (Docker-only)

### Architecture
```
4 Containers:
├── server    (twenty-local:latest, port 3001) - NestJS + React frontend
├── worker    (same image) - BullMQ background jobs
├── db        (postgres:16, port 5432)
└── redis     (redis, port 6379)
```

### Infrastructure
- Application: http://localhost:3001
- PostgreSQL: localhost:5432
- Redis: localhost:6379

---

## Database Safety

### Backup Commands
```bash
# Create backup
docker compose -f docker-compose.local.yml exec db pg_dump -U postgres default | gzip > ~/backup.sql.gz

# Restore backup
gunzip -c ~/backup.sql.gz | docker compose -f docker-compose.local.yml exec -T db psql -U postgres default

# List backups
ls -lht ~/workspace/twentyone/backups/
```

### DANGER - Data Loss Commands
```bash
docker compose -f docker-compose.local.yml down -v   # Deletes all data volumes
npx nx database:reset                                 # Wipes and reseeds database
```

### Safe Commands
```bash
docker compose -f docker-compose.local.yml stop   # Stops containers, keeps data
docker compose -f docker-compose.local.yml down   # Removes containers, keeps volumes
```

---

## Skills

Use `/orchestrator`, `/architect`, `/frontend-dev`, `/backend-dev`, `/qa`, `/devops`

See `.claude/skills/WORKFLOW.md` for details.
