---
name: project-management
description: Manage team projects — repos, cloning, branches, PRs, GitHub identity. Trigger when user mentions project, repo, clone, PR, branch, or GitHub setup.
license: Apache-2.0
compatibility: Requires AI Maestro running on localhost:23000 and gh CLI authenticated.
metadata:
  author: Emasoft
  version: 1.0.0
context: fork
user-invocable: false
---

# Project Management Skill

## Overview

Manage team project infrastructure: GitHub repos, cloning, branches, PRs, and identity.

## Scripts

| Script | Purpose |
|--------|---------|
| `amp-project-info.sh` | Show team/project info |
| `amp-team-members.sh` | List team members with details |
| `amp-project-repos.sh` | List project repositories |
| `amp-clone-repo.sh` | Clone repo to agent work dir |
| `amp-list-local-repos.sh` | List locally cloned repos |
| `amp-create-repo.sh` | Create GitHub repo + register |
| `amp-create-branch.sh` | Create and push a branch |
| `amp-submit-pr.sh` | Create a pull request |
| `amp-task-done.sh` | Report task completion |
| `amp-task-blocked.sh` | Report blocking issue |

## API Endpoints

### GitHub
```
GET  /api/github/auth        — Auth status + project scope check
POST /api/github/auth        — Switch GitHub identity
GET  /api/github/repos       — List repos
POST /api/github/repos       — Create repo
GET  /api/github/orgs        — List organizations
GET  /api/github/projects    — List/validate projects
POST /api/github/projects    — Create project
```

### Team Repos
```
GET  /api/teams/{id}/repos   — List team repos (from GitHub project)
POST /api/teams/{id}/repos   — Register repo with team
```

## Prerequisites

- `gh` CLI installed and authenticated
- For project operations: `gh auth refresh -s project`

## Task Execution Workflow (Programmer)

1. Read task from Orchestrator (via AMP)
2. `amp-project-repos.sh` → identify target repo
3. `amp-list-local-repos.sh` → check if cloned
4. `amp-clone-repo.sh <url>` → clone if needed
5. `amp-create-branch.sh <repo> feature/<desc>`
6. Implement changes (NEVER on main)
7. Run tests
8. `amp-submit-pr.sh <repo> "<title>"`
9. `amp-task-done.sh "PR #N submitted"`
