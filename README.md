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
