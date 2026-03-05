# Email Digital FTE - Bronze Tier

A Digital Full-Time Employee for email management, built with Claude Code, Obsidian, and Gmail MCP.

## What This Is

An AI agent that manages email workflows 24/7 using a three-layer architecture:

| Layer | Tool | Function |
|-------|------|----------|
| Memory | Obsidian Vault | Source of truth you curate |
| Reasoning | Claude Code | Reads memory, follows rules, produces work |
| Action | Gmail MCP | 19 tools for read, send, search, label, filter |

Claude Code without vault = smart agent with amnesia. Claude Code with vault = personal operator that knows your goals, remembers history, follows rules, and builds on previous work.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     USER REQUEST                        │
└────────────────────────┬────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────┐
│              /email-assistant (Master)                   │
│           Interprets intent → Selects workflow           │
└──────┬──────────────┬──────────────────┬────────────────┘
       ▼              ▼                  ▼
┌─────────────┐ ┌───────────────┐ ┌──────────────────────┐
│   SKILLS    │ │  SUBAGENTS    │ │    MCP (Actions)     │
│             │ │               │ │                      │
│ drafter     │ │ inbox-triager │ │ Gmail MCP (19 tools) │
│ templates   │ │ response-     │ │ ├─ send/draft/read   │
│ summarizer  │ │   suggester   │ │ ├─ search/modify     │
│             │ │ follow-up-    │ │ ├─ labels            │
│ (Content    │ │   tracker     │ │ └─ filters           │
│  generation)│ │               │ │                      │
│             │ │ (Classifica-  │ │ Obsidian MCP         │
│             │ │  tion &       │ │ └─ vault read/write  │
│             │ │  judgment)    │ │                      │
└──────┬──────┘ └──────┬────────┘ │ (External data       │
       │               │          │  & actions)          │
       └───────┬───────┘          └──────────┬───────────┘
               ▼                             ▼
┌─────────────────────────────────────────────────────────┐
│                   MEMORY BANK                           │
│                 (Obsidian Vault)                         │
│                                                         │
│  CLAUDE.md ── Entry point & preferences                 │
│  AGENTS.md ── Governance rules                          │
│  references/ ── Tone guidelines, templates, patterns    │
│  agent-memory/ ── Persistent learning across sessions   │
└─────────────────────────────────────────────────────────┘
```

### Orchestration Flow

```
Triage → Suggest → Draft → Send

┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Gmail   │───▶│  Inbox   │───▶│ Response │───▶│  Email   │
│   MCP    │    │  Triager │    │ Suggester│    │  Drafter │
│ (fetch)  │    │ (classify)│   │ (options)│    │ (compose)│
└──────────┘    └──────────┘    └──────────┘    └─────┬────┘
                                                      ▼
                                                ┌──────────┐
                                                │  Gmail   │
                                                │   MCP    │
                                                │ (send)   │
                                                └──────────┘
```

### Delegation Logic

| Request Type | Routes To | Why |
|-------------|-----------|-----|
| Content generation with known patterns | Skills | Templates + tone guidelines |
| Classification requiring judgment | Subagents | Priority assessment, context analysis |
| External data or actions | MCP | Inbox access, sending, labeling |
| Multi-step coordination | Orchestrator | Sequences components end-to-end |

### Four Workflow Modes

| Mode | Trigger | What Happens |
|------|---------|-------------|
| **Inbox Management** | "Triage my inbox" | Fetch unread → Classify priority → Summarize → Suggest responses → Draft replies |
| **Email Composition** | "Write an email to..." | Identify type → Select template → Draft with tone → Create draft or send |
| **Thread Response** | "Help me respond to this thread" | Fetch thread → Summarize → Extract actions → Generate 3 options → Draft selection |
| **Follow-Up Check** | "What emails need follow-up?" | Fetch sent (7-14 days) → Identify awaiting → Flag overdue → Generate follow-ups |

## Components

### Skills (4)

| Skill | Purpose |
|-------|---------|
| `email-assistant` | Master orchestrator - coordinates all components |
| `email-drafter` | Professional email drafting with tone control |
| `email-templates` | Reusable templates with `{{variable}}` substitution |
| `email-summarizer` | Thread analysis extracting decisions, actions, questions |

### Subagents (3)

| Agent | Purpose |
|-------|---------|
| `inbox-triager` | Classifies emails by priority (Urgent/High/Medium/Low) |
| `response-suggester` | Generates reply options with tone labels |
| `follow-up-tracker` | Tracks sent emails awaiting responses |

### MCP Integration

**Gmail MCP** - 19 tools covering:
- Email operations: send, draft, read, search, modify, delete
- Label management: list, create, update, delete
- Filter management: create, list, get, delete

## Directory Structure

```
ai-vault/
├── .claude/
│   ├── skills/
│   │   ├── email-assistant/      # Master orchestrator
│   │   ├── email-drafter/        # Composition skill
│   │   ├── email-templates/      # Template library
│   │   ├── email-summarizer/     # Thread intelligence
│   │   ├── skill-creator/        # Skill authoring tools
│   │   ├── skill-creator-pro/
│   │   └── skill-validator/
│   └── agents/
│       ├── inbox-triager.md
│       ├── response-suggester.md
│       └── follow-up-tracker.md
├── references/
├── CLAUDE.md                     # Project context
├── AGENTS.md                     # Governance rules
└── README.md                     # This file
```

## Prerequisites

- **Claude Code**: Active subscription
- **Obsidian**: v1.10.6+ with Local REST API plugin
- **Gmail MCP**: Configured with OAuth or App Password
- **Python**: 3.13+
- **Node.js**: v24+ LTS

## Setup

### 1. Vault + Memory Bank

```bash
# Clone and navigate
cd ai-vault
```

Obsidian vault serves as the Memory Bank. `CLAUDE.md` is the entry point, `AGENTS.md` defines governance rules.

### 2. Obsidian MCP

```bash
claude mcp add mcp-obsidian --scope user -- uvx mcp-obsidian
```

Configure in `~/.claude.json`:
```json
{
  "mcpServers": {
    "mcp-obsidian": {
      "command": "uvx",
      "args": ["mcp-obsidian"],
      "env": {
        "OBSIDIAN_API_KEY": "your-api-key",
        "OBSIDIAN_HOST": "127.0.0.1",
        "OBSIDIAN_PORT": "27124"
      }
    }
  }
}
```

### 3. Gmail MCP

**Option A: SMTP (quick start)**
```bash
claude mcp add gmail --transport http --scope user \
  https://deep-red-marten.fastmcp.app/mcp \
  --header "X-Gmail-Email: your-email@gmail.com" \
  --header "X-Gmail-Password: xxxx-xxxx-xxxx-xxxx"
```

**Option B: OAuth (full access)** - Recommended for production. Requires Google Cloud project with Gmail API enabled.

### 4. Verify

```bash
claude mcp list
# Expected: mcp-obsidian: connected, gmail: connected
```

## Usage

```bash
cd ai-vault && claude
```

| You Say | Mode | Components Used |
|---------|------|----------------|
| "Help me with email" | Mode menu | Offers all modes |
| "Triage my inbox" | Inbox Management | Gmail MCP + inbox-triager + summarizer |
| "Write a cold outreach" | Composition | Templates + drafter + Gmail MCP |
| "Respond to this thread" | Thread Response | Summarizer + suggester + drafter |
| "Check my follow-ups" | Follow-Up Check | Gmail MCP + follow-up-tracker + templates |

## Graceful Degradation

- **Gmail MCP offline**: Skills work fully, output goes to clipboard instead of Gmail drafts
- **Skill missing**: Falls back to email-drafter for composition
- **Authentication expired**: Guides user through re-authentication

## Security

- Never store credentials in vault files or git
- API keys go in `~/.claude.json` env vars or `.env` (gitignored)
- Always review AI-generated emails before sending
- Draft-first for important external communications

## Quality Gates

Before any email sends:
1. Tone matches guidelines
2. Template variables fully substituted
3. Recipient address verified
4. Draft reviewed if high importance
5. User explicitly confirms send

## Tier Roadmap

| Tier | Adds | Hours |
|------|------|-------|
| **Bronze** (current) | Email assistant, skills, subagents, Gmail MCP | ~4 |
| Silver | Watcher scripts, HITL approval, scheduled ops, CEO briefing | ~8 |
| Gold | Cross-domain integration, multiple MCPs, error recovery, audit logging | ~12+ |
