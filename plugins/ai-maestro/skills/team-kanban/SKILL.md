---
name: team-kanban
description: Manage team kanban boards and tasks — create/update/move/archive tasks, view board status. Trigger when user mentions kanban, tasks, board, cards, columns, or task status.
license: Apache-2.0
compatibility: Requires AI Maestro running on localhost:23000. GitHub Projects require gh auth with project scope.
metadata:
  author: Emasoft
  version: 2.0.0
context: fork
user-invocable: false
---

# Team Kanban Skill

## Overview

Manage team kanban boards backed by GitHub Projects V2. The **ORCHESTRATOR** is the primary kanban manager.

## Scripts (installed at ~/.local/bin/)

| Script | Purpose | Who uses it |
|--------|---------|-------------|
| `amp-kanban-create-task.sh` | Create task (issue + project card) | Orchestrator |
| `amp-kanban-move.sh` | Move card between columns | Orchestrator, COS, Manager |
| `amp-kanban-list.sh` | List tasks with filters | All team members |
| `amp-project-info.sh` | Show team/project info | All |
| `amp-project-repos.sh` | List project repositories | All |

## Standard Columns

| Column | Status Key | Description |
|--------|-----------|-------------|
| Backlog | `backlog` | Future work, not yet prioritized |
| To Do | `todo` | Ready for work |
| In Progress | `in_progress` | Being worked on |
| Review | `review` | PR submitted, awaiting review |
| Done | `done` | Merged and complete |

## Instructions

### Create a task
```bash
amp-kanban-create-task.sh "Implement auth module" --repo owner/repo --assignee agent-name
```

### Move a card
```bash
amp-kanban-move.sh <item-id> in_progress
```

### List tasks
```bash
amp-kanban-list.sh --status todo
amp-kanban-list.sh --assignee agent-name
```

### API Endpoints
```
GET  /api/teams/{id}/kanban/items              — List items
POST /api/teams/{id}/kanban/items              — Create item
PATCH /api/teams/{id}/kanban/items/{itemId}    — Move item
DELETE /api/teams/{id}/kanban/items/{itemId}    — Archive item
```

## Workflow

1. Orchestrator creates tasks from design doc: `amp-kanban-create-task.sh`
2. Orchestrator assigns to agents via AMP: `amp-send.sh <agent> "Task Assignment" "..."`
3. Agent clones repo, creates branch, implements, submits PR
4. Agent reports done: `amp-task-done.sh "PR #N submitted"`
5. Orchestrator moves card: `amp-kanban-move.sh <id> review`
6. Integrator reviews and merges PR
7. Orchestrator moves card: `amp-kanban-move.sh <id> done`
