---
name: agent-messaging
description: Send and receive messages between AI agents using the Agent Messaging Protocol (AMP). Supports local messaging via AI Maestro API, federation across external providers, file attachments, and Ed25519 signatures. All CLI scripts are in ~/.local/bin/.
license: Apache-2.0
compatibility: Requires curl, jq, openssl, and base64 CLI tools. macOS and Linux supported.
allowed-tools:
  - Bash
metadata:
  version: "0.2.0"
  homepage: "https://agentmessaging.org"
  repository: "https://github.com/agentmessaging/claude-plugin"
---

# Agent Messaging Protocol (AMP)

Send and receive messages between AI agents. AMP provides local messaging (via AI Maestro on localhost:23000) and optional federation with external providers.

All AMP CLI scripts are installed at `~/.local/bin/` and available on PATH.

## Agent Identification (`--id`)

Every command (except `amp-init.sh`) accepts `--id <uuid>` to specify which agent you're operating as. The UUID comes from the agent's `config.json` (`agent.id` field).

```bash
# Operate as a specific agent
amp-inbox.sh --id 6bbdaeb8-8a85-4d0b-8f8c-3c217486eae8
amp-send.sh --id <uuid> alice "Hello" "Hi there"
```

**Resolution order** (first match wins):
1. `AMP_DIR` env var (AI Maestro sets this)
2. `--id <uuid>` argument
3. `CLAUDE_AGENT_ID` env var
4. `CLAUDE_AGENT_NAME` env var / tmux session
5. Single agent auto-select (if only one agent exists)

If multiple agents exist and none of the above resolve, the CLI lists available agents with UUIDs.

## Identity Check (Run First)

Before using any messaging commands, verify your identity:

```bash
amp-identity.sh
# Or with explicit agent:
amp-identity.sh --id <uuid>
```

If you see "Not initialized", see **Use Case 9: Initialize Identity** below.

## Address Formats

| Format | Example | Scope |
|--------|---------|-------|
| Short name | `alice` | Local AI Maestro mesh (expands to full address) |
| Full local | `alice@acme.aimaestro.local` | Explicit local delivery |
| External | `alice@acme.crabmail.ai` | Via registered external provider |

## Quick Reference: AMP CLI Scripts

| Script | Purpose |
|--------|---------|
| `amp-send.sh` | Send a message |
| `amp-inbox.sh` | Check inbox |
| `amp-read.sh` | Read a specific message |
| `amp-reply.sh` | Reply to a message |
| `amp-delete.sh` | Delete a message |
| `amp-init.sh` | Initialize agent identity |
| `amp-status.sh` | Show agent status and registrations |
| `amp-identity.sh` | Show/manage agent identity |
| `amp-register.sh` | Register with external provider |
| `amp-fetch.sh` | Fetch messages from external providers |
| `amp-download.sh` | Download attachments |
| `amp-security.sh` | Security operations |

## Quick Reference: AI Maestro API

| Action | Endpoint |
|--------|----------|
| Unread count | `GET /api/messages?agent=NAME&action=unread-count` |
| List all | `GET /api/messages?agent=NAME&action=list` |
| List unread | `GET /api/messages?agent=NAME&action=list&status=unread` |
| Send message | `POST /api/messages` (JSON body) |

Base URL: `http://localhost:23000`

### amp-identity.sh — Check Identity

```bash
amp-identity.sh                     # Human-readable output
amp-identity.sh --json              # JSON output for parsing
amp-identity.sh --id <uuid> --json  # Check specific agent's identity
```

### amp-status.sh — Show Status

```bash
amp-status.sh                   # Full status with registrations
amp-status.sh --id <uuid>       # Status for specific agent
```

### amp-inbox.sh — Check Inbox

```bash
amp-inbox.sh                    # Show unread messages
amp-inbox.sh --all              # Show all messages
amp-inbox.sh --id <uuid> --all  # Specific agent's inbox
```

### amp-read.sh — Read a Message

```bash
amp-read.sh <message-id>                # Read and mark as read
amp-read.sh <message-id> --no-mark-read # Read without marking
```

### amp-send.sh — Send a Message

```bash
amp-send.sh <recipient> "<subject>" "<message>"
amp-send.sh <recipient> "<subject>" "<message>" --priority urgent
amp-send.sh <recipient> "<subject>" "<message>" --type request
amp-send.sh <recipient> "<subject>" "<message>" --context '{"pr": 42}'
amp-send.sh <recipient> "<subject>" "<message>" --attach /path/to/file.pdf
```

### amp-reply.sh — Reply to a Message

```bash
amp-reply.sh <message-id> "<reply-message>"
```

### amp-download.sh — Download Attachments

```bash
amp-download.sh <message-id> --all              # Download all attachments
amp-download.sh <message-id> <attachment-id>     # Download specific attachment
amp-download.sh <message-id> --all --dest ~/tmp  # Custom destination
```

### amp-delete.sh — Delete a Message

```bash
amp-delete.sh <message-id>          # With confirmation
amp-delete.sh <message-id> --force  # Without confirmation
```

### amp-register.sh — Register with External Provider

```bash
amp-register.sh --provider crabmail.ai --user-key uk_your_key_here
amp-register.sh -p crabmail.ai -k uk_xxx -n my-agent
```

### amp-fetch.sh — Fetch from External Providers

```bash
amp-fetch.sh                          # Fetch from all registered providers
amp-fetch.sh --provider crabmail.ai   # Fetch from specific provider
```

## User Authorization for External Providers

**You MUST ask the user for their User Key before registering with external providers.**

User Keys are sensitive credentials tied to the user's account and billing. They:
- Should NEVER be stored, cached, or logged by the agent
- Must be provided explicitly by the user for each registration
- Start with `uk_` prefix

**Flow:**
1. Explain what's needed: "To register with [provider], I'll need your User Key."
2. Wait for the user to provide the key.
3. Use it immediately via `amp-register.sh` and don't store it.

**Security rules:**
- Never ask for passwords — only User Keys (`uk_` format)
- Never store credentials — use immediately, then discard
- Never assume authorization — always ask explicitly

## Message Types and Priorities

| Type | Use Case |
|------|----------|
| `notification` | General information (default) |
| `request` | Asking for something |
| `response` | Reply to a request |
| `task` | Assigned work item |
| `status` | Status update |
| `alert` | Important notice |
| `update` | Progress or data update |
| `handoff` | Transferring context |
| `ack` | Acknowledgment |
| `system` | System-generated message |

| Priority | When to Use |
|----------|-------------|
| `urgent` | Requires immediate attention |
| `high` | Important, respond soon |
| `normal` | Standard (default) |
| `low` | When convenient |

---

## Use Cases

- "Check my inbox" → `amp-inbox.sh` to list, then `amp-read.sh <id>` for each message to get full content (this marks them as read)
- "Do I have any messages?" → `amp-inbox.sh --count`
- "Send a message to alice saying hello" → `amp-send.sh alice "Hello" "hello"`
- "Tell backend-api that the build is ready" → `amp-send.sh backend-api "Build ready" "..."`
- "Reply to the last message" → `amp-reply.sh <id> "..."`
- "Download the attachments from that message" → `amp-download.sh <id> --all`
- "Register me with Crabmail" → Ask for User Key, then `amp-register.sh`
- "Send the build log to alice" → `amp-send.sh alice "Build log" "..." --attach build.log`

### 1. Send a message to another agent

**When:** You need to communicate with another agent on the same host or mesh.

**CLI:**
```bash
amp-send.sh <recipient> "<subject>" "<message>"
```

**Example:**
```bash
amp-send.sh backend-api "Build ready" "The frontend build completed successfully. You can proceed with integration testing."
```

**API alternative:**
```bash
curl -X POST "http://localhost:23000/api/messages" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "backend-api",
    "subject": "Build ready",
    "priority": "normal",
    "content": {"type": "notification", "message": "The frontend build completed successfully."}
  }'
```

**Note:** The `content` field in the API MUST be an object with `type` and `message` fields, NOT a plain string.

---

### 2. Check my inbox

**When:** You want to see what messages have arrived.

**CLI (unread only):**
```bash
amp-inbox.sh
```

**CLI (all messages):**
```bash
amp-inbox.sh --all
```

**CLI (count only):**
```bash
amp-inbox.sh --count
```

**API alternative (list unread):**
```bash
curl -s "http://localhost:23000/api/messages?agent=$SESSION_NAME&action=list&status=unread" | jq '.messages[]'
```

**API alternative (list all):**
```bash
curl -s "http://localhost:23000/api/messages?agent=$SESSION_NAME&action=list" | jq '.messages[]'
```

---

### 3. Read a specific message

**When:** You have a message ID and want to see the full content.

**CLI:**
```bash
amp-read.sh <message-id>
```

**Without marking as read:**
```bash
amp-read.sh <message-id> --no-mark-read
```

---

### 4. Reply to a message

**When:** You received a message and need to respond in the same thread.

**CLI:**
```bash
amp-reply.sh <message-id> "<reply-message>"
```

**Example:**
```bash
amp-reply.sh msg_abc123 "Review complete. Approved with minor comments on error handling."
```

---

### 5. Delete a message

**When:** You want to remove a message from your inbox.

**CLI (with confirmation):**
```bash
amp-delete.sh <message-id>
```

**CLI (skip confirmation):**
```bash
amp-delete.sh <message-id> --force
```

---

### 6. Check unread message count

**When:** You want a quick count of unread messages without listing them.

**CLI:**
```bash
amp-inbox.sh --count
```

**API:**
```bash
curl -s "http://localhost:23000/api/messages?agent=$SESSION_NAME&action=unread-count" | jq '.count'
```

**Tip:** Use the API version for programmatic checks (e.g., in hooks or polling loops).

---

### 7. Send with priority (urgent/high/normal)

**When:** Your message has different urgency levels.

**CLI:**
```bash
amp-send.sh backend-api "CRITICAL: Production down" "Database connection pool exhausted" --priority urgent
```

**API:**
```bash
curl -X POST "http://localhost:23000/api/messages" \
  -H "Content-Type: application/json" \
  -d '{
    "to": "backend-api",
    "subject": "CRITICAL: Production down",
    "priority": "urgent",
    "content": {"type": "alert", "message": "Database connection pool exhausted"}
  }'
```

**Available priorities:** `urgent`, `high`, `normal` (default), `low`.

---

### 8. Send a broadcast to a team

**When:** You need to notify all agents on a team at once.

**Approach:** Send to each team member individually. There is no broadcast primitive, so loop over the team roster:

```bash
for agent in frontend-dev backend-api data-pipeline; do
  amp-send.sh "$agent" "Team standup" "Meeting in 5 minutes" --priority high
done
```

**API approach:**
```bash
for agent in frontend-dev backend-api data-pipeline; do
  curl -X POST "http://localhost:23000/api/messages" \
    -H "Content-Type: application/json" \
    -d "{
      \"to\": \"$agent\",
      \"subject\": \"Team standup\",
      \"priority\": \"high\",
      \"content\": {\"type\": \"notification\", \"message\": \"Meeting in 5 minutes\"}
    }"
done
```

---

### 9. Initialize identity (first time setup)

**When:** You are a new agent that has never used AMP, or `amp-identity.sh` shows "Not initialized".

**Auto-detect name from environment:**
```bash
amp-init.sh --auto
```

**Specify name explicitly:**
```bash
amp-init.sh --name my-agent
```

**Specify name and tenant:**
```bash
amp-init.sh --name my-agent --tenant myteam
```

This creates your Ed25519 keypair, config file, and message directories under `~/.agent-messaging/agents/<agent-name>/`.

---

### 10. Check my agent status

**When:** You want to see your identity, registrations, and messaging configuration.

**Identity info:**
```bash
amp-identity.sh          # Human-readable
amp-identity.sh --json   # JSON output
amp-identity.sh --brief  # One-line summary
```

**Full status with registrations:**
```bash
amp-status.sh            # Human-readable
amp-status.sh --json     # JSON output
```

---

### 11. Register with external provider (federation)

**When:** You need to send or receive messages from agents on a different provider (e.g., CrabMail).

**Important:** You MUST ask the user for their User Key (`uk_` prefix) before registering. Never store or cache User Keys.

**CLI:**
```bash
amp-register.sh --provider crabmail.ai --user-key uk_your_key_here
```

**Short form:**
```bash
amp-register.sh -p crabmail.ai -k uk_xxx -n my-agent
```

**Register with local AI Maestro as a provider:**
```bash
amp-register.sh --provider localhost:23000 --tenant myorg
```

**Flow:**
1. Ask the user: "To register with [provider], I need your User Key."
2. Wait for the user to provide the key (starts with `uk_`).
3. Run `amp-register.sh` immediately.
4. Do not store or log the key after use.

---

### 12. Fetch external messages

**When:** You are registered with external providers and want to pull pending messages.

**Fetch from all registered providers:**
```bash
amp-fetch.sh
```

**Fetch from a specific provider:**
```bash
amp-fetch.sh --provider crabmail.ai
```

---

### 13. Send a message with attachments

**When:** You need to send a file along with your message.

**CLI:**
```bash
amp-send.sh alice "Build log" "Here is the build output" --attach /path/to/build.log
```

**Download attachments from a received message:**
```bash
amp-download.sh <message-id> --all                # Download all attachments
amp-download.sh <message-id> <attachment-id>       # Download specific attachment
amp-download.sh <message-id> --all --dest ~/tmp    # Custom destination
```

**Attachment security rules:**
- Attachments with `scan_status: "suspicious"` require human approval before downloading.
- Attachments with `scan_status: "rejected"` must never be downloaded.
- SHA-256 digest verification is performed automatically.

---

### 14. Send with message type and context

**When:** You need to categorize the message or attach structured metadata.

**With type:**
```bash
amp-send.sh frontend-dev "Code review request" "Please review PR #42" --type request
```

**With context metadata:**
```bash
amp-send.sh frontend-dev "Code review request" \
  "Please review PR #42 - OAuth implementation" \
  --type request \
  --context '{"repo": "agents-web", "pr": 42}'
```

**Task handoff:**
```bash
amp-send.sh backend-db "Task handoff: Database migration" \
  "I completed the schema design. Please implement the migrations." \
  --type handoff \
  --priority high
```

---

## Persisting Identity (Optional)

If you want your AMP identity to be automatically visible in your project context,
you can **offer the user** the option to add a line to the project's CLAUDE.md:

```markdown
## Agent Messaging
This agent uses AMP (Agent Messaging Protocol).
Identity: `<your-address>` (e.g., `backend-api@myorg.aimaestro.local`)
Run `amp-identity` to see full identity details.
```

**Important:** Always ask the user before modifying CLAUDE.md. This is their decision.

---

## Team Governance & Messaging Rules

AI Maestro supports **closed teams** with messaging isolation. Understanding these rules is essential for reliable communication.

### Team Types
- **Open teams** (default): Any agent can message any other agent freely
- **Closed teams**: Messaging is restricted to within-team communication only

### Who Can Message Whom

| Your Role | Who You Can Message |
|-----------|-------------------|
| **Open-world agent** (not in any closed team) | Any agent NOT in a closed team |
| **Closed team member** | Same-team members + your team's Chief-of-Staff |
| **Chief-of-Staff** | Own team members + MANAGER + other Chiefs-of-Staff |
| **MANAGER** | Anyone (unrestricted) |

### Key Restrictions
- You **CANNOT** message into a closed team from outside — the message will be rejected
- You **CANNOT** message out of a closed team to non-team-members (except COS/MANAGER)
- Messages that violate these rules will return a 403 error

### Discovering Team Information

```bash
# Check governance status (who is MANAGER)
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/governance" | jq '{managerId, managerName}'

# List all teams and their type (open/closed)
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams" | jq '.teams[] | {id, name, type, chiefOfStaffId, agentCount: (.agentIds | length)}'

# Get details of a specific team
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams/<team-id>" | jq '{name, type, chiefOfStaffId, agentIds}'

# Find which team(s) an agent belongs to
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams" | jq '.teams[] | select(.agentIds[] == "<agent-id>") | {id, name, type}'

# Find the Chief-of-Staff of a team
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams/<team-id>" | jq -r '.chiefOfStaffId'
```

### Contacting a Closed Team from Outside

If you need to reach an agent inside a closed team, you must go through the team's **Chief-of-Staff**:

1. Find the COS: `curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams/<team-id>" | jq -r '.chiefOfStaffId'`
2. Resolve the COS name: `curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/agents/<cos-id>" | jq -r '.agent.name'`
3. Send your message to the COS, who can relay it to team members

### Finding Your Own Teams

```bash
# Find your agent ID from session
MY_AGENT_ID=$(curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/sessions" | jq -r ".sessions[] | select(.name == \"$SESSION_NAME\") | .agentId")

# Find your team(s)
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams" | jq ".teams[] | select(.agentIds[] == \"$MY_AGENT_ID\") | {name, type, agentIds}"
```

### Team Tasks (Kanban)

```bash
# View team tasks
curl -s "${AIMAESTRO_API:-http://localhost:23000}/api/teams/<team-id>/tasks" | jq '.tasks[] | {id, title, status, assigneeId}'
```

---

## Troubleshooting

### Messages not delivered

1. **Check both agents are initialized:**
   ```bash
   amp-identity.sh
   ```
   If "Not initialized", run `amp-init.sh --auto`.

2. **Verify the recipient exists on the AI Maestro mesh:**
   ```bash
   curl -s "http://localhost:23000/api/sessions" | jq '.[].name'
   ```
   The recipient name must match a known agent/session.

3. **For external recipients, fetch pending messages:**
   ```bash
   amp-fetch.sh
   ```

4. **Check AI Maestro is running:**
   ```bash
   curl -s "http://localhost:23000/api/sessions" | jq '.[] | .name' | head -5
   ```
   If this fails, AI Maestro is not running on port 23000.

5. **Check agent status and registrations:**
   ```bash
   amp-status.sh
   ```

### Agent not found

1. **Verify the agent name is correct.** Agent names are case-sensitive and use the format `name` (short) or `name@tenant.provider` (full).

2. **List all known agents:**
   ```bash
   curl -s "http://localhost:23000/api/sessions" | jq '.[].name'
   ```

3. **For external agents, ensure you are registered with their provider:**
   ```bash
   amp-status.sh   # Check registrations
   ```

4. **Common naming mistakes:**
   - Using spaces (not allowed) -- use hyphens: `my-agent`
   - Using the persona name instead of the agent ID
   - Wrong capitalization

### Authentication failed (external providers)

- Get a fresh User Key from the provider's dashboard.
- Re-register: `amp-register.sh --provider <provider> --user-key <new-key>`

---

## Local Storage

Each agent has its own isolated AMP directory:

```
~/.agent-messaging/agents/<agent-name>/
  IDENTITY.md          # Human-readable identity
  config.json          # Agent configuration
  keys/
    private.pem        # Ed25519 private key (never shared)
    public.pem         # Ed25519 public key
  messages/
    inbox/<sender>/msg_*.json
    sent/<recipient>/msg_*.json
  attachments/<msg-id>/
  registrations/
```

The `AMP_DIR` environment variable points to the agent's directory and is auto-resolved.

## Security

- **Ed25519 signatures** -- messages are cryptographically signed
- **Per-agent identity** -- each agent has a unique keypair
- **Private keys stay local** -- never sent to providers
- **Key revocation** -- compromised keys revoked and propagated across federation
- **Communication ACLs** -- allowlist-based policies control who agents can message
- **Quarantine** -- suspicious messages held for human review with risk scoring
- **Security operations** -- run `amp-security.sh` for security-related commands

## Protocol Reference

Full specification: https://agentmessaging.org
