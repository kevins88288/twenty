# Agent Documentation Index

Documentation for the High Signal VC CRM project built on Twenty CRM.

## Quick Links

| Document | Purpose |
|----------|---------|
| [architecture.md](architecture.md) | System design, deployment, tech stack |
| [datamodel.md](datamodel.md) | Entity reference, relationships, status values |
| [prd.md](prd.md) | Product requirements, user stories, success metrics |
| [roadmap.md](roadmap.md) | Development phases, task breakdown |
| [twentypattern.md](twentypattern.md) | Twenty workspace entity patterns (READ FIRST) |
| [coding-standards.md](coding-standards.md) | All coding conventions, style guides |
| [dev_log.md](dev_log.md) | Development journal, decisions, learnings |

## How to Use These Docs

1. **Start with** `prd.md` to understand requirements
2. **Review** `architecture.md` for system design
3. **Follow** `twentypattern.md` when creating entities
4. **Reference** `coding-standards.md` for conventions
5. **Check** `roadmap.md` for current phase
6. **Log decisions** in `dev_log.md`

## Quick Commands

See `CLAUDE.md` in project root for:
- Development commands (twenty-dev, twenty-docker)
- OCI deployment details
- Database backup commands
- Skills reference

## Documentation Ownership

Each skill is responsible for updating relevant docs:

| Skill | Primary Docs |
|-------|--------------|
| architect | architecture.md, datamodel.md |
| backend-dev | twentypattern.md, coding-standards.md (server) |
| frontend-dev | coding-standards.md (react, state) |
| qa | coding-standards.md (testing) |
| devops | architecture.md (deployment) |
| orchestrator | roadmap.md (progress) |

See `.claude/skills/WORKFLOW.md` for full documentation workflow.
