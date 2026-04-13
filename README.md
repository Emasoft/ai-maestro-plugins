# AI Maestro Plugins Marketplace

Official plugin marketplace for [AI Maestro](https://github.com/Emasoft/ai-maestro).

This repo is a **plugin index** — it lists available plugins that Claude Code can install. The plugins themselves live in their own repos.

## Installing Plugins

```bash
# Install the core AI Maestro plugin (11 skills, 12 commands)
claude plugin install ai-maestro-plugin ai-maestro-plugins --scope user

# Install a role-plugin (installs to agent's project scope)
claude plugin install ai-maestro-architect-agent ai-maestro-plugins --scope local
```

## Available Plugins

### Core Plugin

| Plugin | Description | Version |
|--------|-------------|---------|
| [ai-maestro-plugin](https://github.com/Emasoft/ai-maestro-plugin) | Agent management, AMP messaging, AID identity, memory search, code graph, docs search, planning, team governance, team kanban, MCP discovery, hook debugging | 2.3.1 |

### Role-Plugins

| Plugin | Governance Title | Description |
|--------|-----------------|-------------|
| [ai-maestro-assistant-manager-agent](https://github.com/Emasoft/ai-maestro-assistant-manager-agent) | MANAGER | User interlocutor, directs other agents |
| [ai-maestro-chief-of-staff](https://github.com/Emasoft/ai-maestro-chief-of-staff) | CHIEF-OF-STAFF | Per-team agent management, staff planning |
| [ai-maestro-architect-agent](https://github.com/Emasoft/ai-maestro-architect-agent) | ARCHITECT | Design documents, requirements, architecture |
| [ai-maestro-orchestrator-agent](https://github.com/Emasoft/ai-maestro-orchestrator-agent) | ORCHESTRATOR | Task distribution, kanban, coordination |
| [ai-maestro-integrator-agent](https://github.com/Emasoft/ai-maestro-integrator-agent) | INTEGRATOR | Quality gates, PR review, merging, releases |
| [ai-maestro-programmer-agent](https://github.com/Emasoft/ai-maestro-programmer-agent) | MEMBER | General-purpose implementer, writes code |

## License

MIT

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
