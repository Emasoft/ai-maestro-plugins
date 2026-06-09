# AI Maestro Plugins Marketplace

Official plugin marketplace for [AI Maestro](https://github.com/Emasoft/ai-maestro).

This repo is a **plugin index** — it lists available plugins that Claude Code can install. The plugins themselves live in their own repos. Plugin versions are tracked in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json) and bump automatically when an upstream plugin tags a release, so the tables below intentionally omit version numbers.

## Available Plugins

### Core Plugins

Install with `--scope user` (all sessions) or `--scope project`.

| Plugin | Category | Description |
|--------|----------|-------------|
| [ai-maestro-plugin](https://github.com/Emasoft/ai-maestro-plugin) | productivity | Agent management, memory search, code graph, AMP messaging, AID identity, docs search, planning, team governance, team kanban, MCP discovery, hook debugging, network security. Requires the AI Maestro server. |
| [ai-maestro-janitor](https://github.com/Emasoft/ai-maestro-janitor) | core-plugin | Session-scoped janitor — reconciles PRs, worktrees, TRDD drift, and task/PR mismatches, auto-resumes on rate-limit, keeps the prompt cache warm via a single durable cron heartbeat. No external daemon. |

### Role-Plugins

One per AI Maestro governance title. Install with `--scope local` inside the agent's project directory.

| Plugin | Governance Title | Description |
|--------|-----------------|-------------|
| [ai-maestro-assistant-manager-agent](https://github.com/Emasoft/ai-maestro-assistant-manager-agent) | MANAGER | User interlocutor, directs other agents |
| [ai-maestro-chief-of-staff](https://github.com/Emasoft/ai-maestro-chief-of-staff) | CHIEF-OF-STAFF | Per-team agent management, staff planning |
| [ai-maestro-architect-agent](https://github.com/Emasoft/ai-maestro-architect-agent) | ARCHITECT | Design documents, requirements, architecture |
| [ai-maestro-orchestrator-agent](https://github.com/Emasoft/ai-maestro-orchestrator-agent) | ORCHESTRATOR | Task distribution, kanban, coordination |
| [ai-maestro-integrator-agent](https://github.com/Emasoft/ai-maestro-integrator-agent) | INTEGRATOR | Quality gates, PR review, merging, releases |
| [ai-maestro-programmer-agent](https://github.com/Emasoft/ai-maestro-programmer-agent) | PROGRAMMER | General-purpose implementer, writes code |
| [ai-maestro-maintainer-agent](https://github.com/Emasoft/ai-maestro-maintainer-agent) | MAINTAINER | Polls GitHub issues, triages bugs, fixes valid issues via publish pipeline |
| [ai-maestro-autonomous-agent](https://github.com/Emasoft/ai-maestro-autonomous-agent) | AUTONOMOUS | Mandatory role-plugin for no-team agents; enforces workspace isolation, forbids cross-agent mutation, respects the AMP comm graph |

### Visualization

| Plugin | Description |
|--------|-------------|
| [ai-maestro-visual-communicator-plugin](https://github.com/Emasoft/ai-maestro-visual-communicator-plugin) | Interactive HTML pages — diagrams, diff reviews, plan reviews, slide decks, data tables, and modal-comment agent reports. Every page sends the user's selection back to the agent. |

## Installation

Add this marketplace to Claude Code, then install individual plugins:

```bash
# Register this marketplace (one-time setup)
claude plugin marketplace add https://github.com/Emasoft/ai-maestro-plugins

# Install the core AI Maestro plugin (user scope -- available in all sessions)
claude plugin install ai-maestro-plugin ai-maestro-plugins --scope user

# Install a role-plugin (local scope -- per agent project directory)
claude plugin install ai-maestro-programmer-agent ai-maestro-plugins --scope local

# Restart Claude Code to activate newly installed plugins
# In Claude Code: use /reload-plugins or restart the session
```

## Update

To update all plugins from this marketplace:

```bash
# Update the marketplace index
claude plugin marketplace update ai-maestro-plugins

# Update a specific plugin
claude plugin update ai-maestro-plugin@ai-maestro-plugins
claude plugin update ai-maestro-programmer-agent@ai-maestro-plugins
```

## Uninstall

To remove individual plugins:

```bash
# Uninstall the core plugin (user scope)
claude plugin uninstall ai-maestro-plugin --scope user

# Uninstall a role-plugin (local scope)
claude plugin uninstall ai-maestro-programmer-agent --scope local

# Remove the marketplace registration
claude plugin marketplace remove ai-maestro-plugins
```

## Troubleshooting

**Plugin not found after marketplace add**
Run `claude plugin marketplace update ai-maestro-plugins` to refresh the index, then retry the install.

**Plugin not found with the name from its repo URL**
Install names come from each plugin's `.claude-plugin/plugin.json` `name` field (mirrored in this repo's `marketplace.json`), which is not always identical to the GitHub repo name. Check the tables above for the exact install name.

**Role-plugin not activating**
Role-plugins must be installed with `--scope local` inside the agent's project directory. Verify you are in the correct working directory before installing.

**Version not updating after plugin release**
Run `claude plugin marketplace update ai-maestro-plugins` to pull the latest marketplace.json. If the version is still stale, try `claude plugin update <plugin-name>@ai-maestro-plugins`.

**Hook path not found after update**
After updating a plugin, restart Claude Code. If hook paths still cannot be found, run `claude plugin uninstall <plugin>` and reinstall from scratch.

**Old version shown after update**
Claude Code caches plugin files. Run `claude plugin marketplace update ai-maestro-plugins` to refresh the index, then `claude plugin update <plugin-name>@ai-maestro-plugins`. A Claude Code restart is required to pick up the new version.

**Restart required after install or update**
Plugin changes (skills, hooks, commands) take effect only after restarting Claude Code or running `/reload-plugins` in the active session.

**MARKETPLACE_PAT errors in CI**
The notify-marketplace workflow in each plugin repo requires a PAT secret named `MARKETPLACE_PAT`. See each plugin repo's `.github/workflows/notify-marketplace.yml` setup comments for how to create and store the token.

## License

MIT
