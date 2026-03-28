---
name: team-governance
description: Manage team governance — create/delete teams, assign titles (MANAGER, CHIEF-OF-STAFF, ORCHESTRATOR, MEMBER), set passwords, transfer agents, and check access. Trigger when user mentions teams, titles, governance, COS, orchestrator, or transfers.
license: Apache-2.0
compatibility: Requires AI Maestro running on localhost:23000
metadata:
  author: Emasoft
  version: 2.0.0
context: fork
user-invocable: false
---

# Team Governance Skill

## Overview

Manage AI Maestro governance: teams, titles, passwords, transfers, and access control.

## Governance Titles (4 levels)

| Title | Description | Plugin |
|-------|-------------|--------|
| **MANAGER** | Singleton. Full authority over all teams and agents. | ai-maestro-assistant-manager-agent |
| **CHIEF-OF-STAFF** | Leads a team. Manages membership, routes external messages. | ai-maestro-chief-of-staff |
| **ORCHESTRATOR** | Primary kanban manager. Direct MANAGER communication. | ai-maestro-orchestrator-agent |
| **MEMBER** | Default. Can message teammates + COS + Orchestrator + Manager. | (any role-plugin) |

## Instructions

### Team Operations

```bash
# Create team (requires password)
curl -X POST http://localhost:23000/api/teams \
  -H "Content-Type: application/json" \
  -d '{"name": "Backend Squad", "password": "...", "chiefOfStaffId": "<agent-uuid>"}'

# Delete team
curl -X DELETE http://localhost:23000/api/teams/<teamId> \
  -H "Authorization: Bearer <password>"

# Update team
curl -X PUT http://localhost:23000/api/teams/<teamId> \
  -H "Content-Type: application/json" \
  -d '{"name": "New Name", "description": "Updated"}'
```

### Title Assignment

```bash
# Set MANAGER (singleton — replaces existing)
curl -X POST http://localhost:23000/api/governance/manager \
  -H "Content-Type: application/json" \
  -d '{"agentId": "<uuid>", "password": "..."}'

# Set CHIEF-OF-STAFF for a team
curl -X POST http://localhost:23000/api/teams/<teamId>/chief-of-staff \
  -H "Content-Type: application/json" \
  -d '{"agentId": "<uuid>", "password": "..."}'

# Set ORCHESTRATOR for a team
curl -X PUT http://localhost:23000/api/teams/<teamId> \
  -H "Content-Type: application/json" \
  -d '{"orchestratorId": "<uuid>"}'
```

### Messaging Rules

- **MANAGER** → can message anyone
- **CHIEF-OF-STAFF** → can message team members + MANAGER + other COS
- **ORCHESTRATOR** → can message team members + MANAGER directly (kanban coordination)
- **MEMBER** → can message teammates + COS + Orchestrator + Manager

### Agent Transfers

```bash
# Request transfer (COS initiates)
curl -X POST http://localhost:23000/api/v1/governance/requests \
  -H "Content-Type: application/json" \
  -d '{"type": "transfer", "agentId": "<uuid>", "fromTeamId": "...", "toTeamId": "...", "password": "..."}'
```

## Key Rules

- All teams are **closed** (isolated messaging through COS)
- Each agent belongs to **at most one team**
- COS **must** have a team. MANAGER is usually teamless.
- ORCHESTRATOR is the primary kanban manager for the team
- Team creation auto-creates COS
- Title changes require governance password
