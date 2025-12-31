# Twenty CRM Development Workflow

This document explains how to use Claude Code's agent skills for Twenty CRM development.

## Available Agents

| Agent | Slash Command | When to Use |
|-------|---------------|-------------|
| **Orchestrator** | `/orchestrator` | Complex features requiring multiple specialists |
| **Architect** | `/architect` | System design, data models, API contracts |
| **Frontend Dev** | `/frontend-dev` | React components, Recoil state, UI work |
| **Backend Dev** | `/backend-dev` | NestJS APIs, GraphQL, database work |
| **DevOps** | `/devops` | Build issues, Docker, CI/CD |
| **QA** | `/qa` | Testing, coverage, E2E tests |

## How Skills Work

Skills are **automatically discovered** by Claude based on your request. You can also invoke them explicitly:

```
/architect design a notification system for Twenty
```

Or let Claude choose automatically:

```
I need to add a notes field to the Company entity with full CRUD support
```

Claude will recognize this needs architect → backend → frontend → qa coordination.

## Workflow Examples

### Example 1: New Feature (Full Stack)

**Request:**
```
Add a tagging system for companies where users can create, assign, and filter by tags
```

**Workflow:**
1. Claude invokes **orchestrator** to plan
2. **Architect** designs: data model, API, component structure
3. **Backend-dev** implements: migration, entity, resolver
4. **Frontend-dev** builds: tag components, state, queries
5. **QA** verifies: unit tests, E2E tests

### Example 2: Frontend Only

**Request:**
```
Improve the company list performance - it's slow with 1000+ items
```

**Workflow:**
1. Claude invokes **frontend-dev** directly
2. Analyzes current implementation
3. Implements virtualization or pagination
4. Adds performance tests

### Example 3: Backend Only

**Request:**
```
Add rate limiting to the GraphQL API
```

**Workflow:**
1. Claude invokes **backend-dev** directly
2. Designs rate limiting strategy
3. Implements NestJS guard/middleware
4. Adds integration tests

### Example 4: Bug Fix

**Request:**
```
The calendar sync is failing for Google accounts
```

**Workflow:**
1. Claude analyzes which specialist is needed
2. Invokes **backend-dev** (likely API issue)
3. Debugs and fixes the issue
4. **QA** adds regression test

### Example 5: Infrastructure

**Request:**
```
The CI pipeline is taking 20 minutes, optimize it
```

**Workflow:**
1. Claude invokes **devops** directly
2. Analyzes current Nx configuration
3. Optimizes caching and parallelization
4. Verifies improvement

## Best Practices

### 1. Start with Clear Requirements

**Good:**
```
Add a dashboard widget that shows the top 10 companies by revenue with a bar chart
```

**Too vague:**
```
Add some charts
```

### 2. Let Orchestrator Handle Complex Work

For features touching multiple layers:
```
/orchestrator implement user activity tracking that logs page views,
button clicks, and API calls, with a dashboard to view the data
```

### 3. Go Direct for Focused Tasks

For single-layer work:
```
/frontend-dev fix the date picker component timezone handling
```

```
/backend-dev add pagination to the GET /companies endpoint
```

### 4. Use Plan Mode for Architecture

For significant features, start with planning:
```
/plan design a workflow automation system for Twenty
```

This uses Claude's plan mode to explore the codebase and propose an approach before coding.

### 5. Coordinate Testing

After implementation:
```
/qa add E2E tests for the new tagging feature
```

## Quick Reference

### Starting Development
```bash
# Terminal 1: Start services
docker compose up -d
npx nx start

# Terminal 2: Claude Code
claude

# In Claude Code
What Skills are available?
```

### Common Commands Within Claude

```
# Feature development
/orchestrator add [feature description]

# Architecture review
/architect evaluate the current calendar module design

# Frontend work
/frontend-dev build a [component] for [purpose]

# Backend work
/backend-dev implement [API/feature]

# Testing
/qa add tests for [feature]

# Infrastructure
/devops fix [build/deploy issue]
```

### Verifying Changes

```
# Run all checks
npx nx run-many -t lint,typecheck,test

# Run specific package
npx nx test twenty-front
npx nx test twenty-server

# Run E2E
npx nx e2e twenty-e2e-testing
```

## Troubleshooting

### Skills Not Found

Ensure you're in the twentyone directory:
```bash
cd /home/ubuntu/workspace/twentyone
claude
```

### Skill Not Triggering

Invoke explicitly:
```
/frontend-dev [your request]
```

### Need More Context

Ask Claude to explore first:
```
Explore the calendar module and explain how Google sync works
```

Then request the change:
```
Now add retry logic for failed sync attempts
```

---

## Documentation Workflow

### After Each Task

1. Update relevant `.agent_docs/` files
2. Add entry to `dev_log.md`
3. Update `roadmap.md` task status if applicable

### Documentation Ownership

| Skill | Primary Docs |
|-------|--------------|
| architect | architecture.md, datamodel.md |
| backend-dev | twentypattern.md, coding-standards.md (server) |
| frontend-dev | coding-standards.md (react, state) |
| qa | coding-standards.md (testing) |
| devops | architecture.md (deployment) |
| orchestrator | roadmap.md (progress) |

### dev_log.md Entry Format

```markdown
## YYYY-MM-DD

### [Type]: [Title]
**Status**: Completed | In Progress
**Summary**: [One-sentence description]
**Changes**: [List of changes]
**Files**: [List of affected files]
```

Types: Architecture Decision, Implementation, Bug Fix, Learning, Documentation

### Feedback Loop

When user provides feedback revealing patterns:
1. Update relevant `coding-standards.md` section
2. Log the learning in `dev_log.md`
3. Apply to current task
