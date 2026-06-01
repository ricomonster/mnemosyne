# mnemosyne

Mnemosyne is a set of configuration files for [Claude Code](https://docs.anthropic.com/claude-code) and [OpenCode](https://opencode.ai) that enforces a structured, agent-based workflow.

## What it is

Mnemosyne provides configuration layers for both Claude Code (`.claude/`) and OpenCode (`.opencode/` or `~/.config/opencode/`) that define a set of roles, boundaries, and escalation rules. Instead of the assistant acting as a generalist, it routes tasks to specialized agents — each with a fixed model, a narrow scope, and explicit limits on what it can do.

The default mode is **read-only analysis**: the assistant researches, reasons, and advises. All execution (file changes, git operations) is performed explicitly by the user or triggered through the release-engineer agent.

---

## Structure

.
├── claude/                     # Claude Code configuration
│   ├── CLAUDE.md               # Global instructions, agent routing, workflows
│   ├── settings.json           # Permission rules, default model, themes
│   └── agents/
│       ├── junior-engineer.md
│       ├── senior-engineer.md
│       ├── architect.md
│       ├── principal-engineer.md
│       └── release-engineer.md
├── opencode/                   # OpenCode configuration
│   ├── opencode.jsonc          # Agent definitions, permissions, model assignments
│   ├── AGENTS.md               # Global instructions for OpenCode
│   └── agents/
│       ├── junior-engineer.md
│       ├── senior-engineer.md
│       ├── architect.md
│       ├── principal-engineer.md
│       └── release-engineer.md
├── LICENSE                     # MIT License
└── README.md

---

## Operating mode

- Analysis only by default: no file creation, modification, or deletion
- No unsolicited offers to implement ("shall I...", "should I...")
- All mutations are user-initiated or go through `release-engineer`
- Bash access is read-only: `git log/diff/status`, `grep`, test runners

---

## Agents

### Claude Code

| Agent | Model | Role |
|---|---|---|
| `junior-engineer` | `claude-haiku-4-5-20251001` | Repository exploration: find, list, grep, trace symbols |
| `senior-engineer` | `claude-sonnet-4-6` | Implementation, debugging, refactoring, code review |
| `architect` | `claude-opus-4-7` | System design, API contracts, tradeoff analysis, pre-implementation planning |
| `principal-engineer` | `claude-opus-4-7` | System-wide risk, scalability, long-term architectural validation |
| `release-engineer` | `claude-sonnet-4-6` | Git operations: stage, commit, push — with safety checks |

### OpenCode

| Agent | Model | Role |
|---|---|---|
| `junior-engineer` | `opencode/qwen3.6-plus` | Read-only repository exploration |
| `senior-engineer` | `opencode/qwen3.6-plus` | Implementation, debugging, refactoring (has edit permission) |
| `architect` | `opencode/kimi-k2.6` | System design and planning (read-only) |
| `principal-engineer` | `opencode/glm-5.1` | System-wide risk analysis (read-only) |
| `release-engineer` | `opencode/qwen3.6-plus` | Git operations with safety checks (read-only except git) |

### Boundaries

**junior-engineer** — observes only. No quality judgments, no suggestions, no design inference.

**senior-engineer** — builds and fixes. Does not decide service boundaries or system architecture.

**architect** — designs before code is written. Does not write production code or debug.

**principal-engineer** — evaluates risk at the system level. Not part of the default workflow. Only invoked when genuine system-wide concerns are present.

**release-engineer** — git operations only. Requires explicit user confirmation before commit and push. Never runs destructive commands.

---

## Default workflow

For reviews or unscoped requests:

1. `junior-engineer` — map relevant code and structure
2. `senior-engineer` — implementation-level analysis or fix
3. `architect` — only if design decisions need validation
4. `principal-engineer` — only if system-level risk is detected

Skip any step that is not needed.

### Release workflow

Triggered by: `commit`, `push`, `ship`, `release`, `stage`

1. `release-engineer` runs `git status` and `git diff`
2. Scans for sensitive files before staging
3. Generates a conventional commit message (`type(scope): description`)
4. Shows staging plan and commit message — waits for confirmation
5. Pushes only when explicitly requested

---

## Setup

### Prerequisites

- [Claude Code](https://docs.anthropic.com/claude-code) installed
- [OpenCode](https://opencode.ai) installed
- Claude Max or API access with model permissions for Haiku, Sonnet, and Opus

### Installation

#### Claude Code

Copy the contents into `~/.claude/`:

```sh
mkdir -p ~/.claude/agents
cp claude/CLAUDE.md ~/.claude/CLAUDE.md
cp claude/settings.json ~/.claude/settings.json
cp claude/agents/* ~/.claude/agents/
```

Claude Code loads `.claude/CLAUDE.md` and `.claude/settings.json` automatically when present.

#### OpenCode

Copy the contents into `~/.config/opencode/`:

```sh
mkdir -p ~/.config/opencode/agents
cp opencode/opencode.jsonc ~/.config/opencode/opencode.jsonc
cp opencode/AGENTS.md ~/.config/opencode/AGENTS.md
cp opencode/agents/* ~/.config/opencode/agents/
```

OpenCode loads `~/.config/opencode/opencode.jsonc` automatically when present.

### What gets installed
- CLAUDE.md / AGENTS.md: Operating mode, agent routing rules, and workflow definitions
- settings.json / opencode.jsonc: Permission rules, model assignments, agent definitions
- agents/: Individual agent definition files with role-specific boundaries

### Verify

#### Claude Code
Open Claude Code in any project and confirm:
1. Analysis-only mode is active (file edit operations require explicit approval)
2. Agent list appears with `/agents` command
You should see `junior-engineer`, `senior-engineer`, `architect`, `principal-engineer`, and `release-engineer`.

#### OpenCode
Run opencode in any project and confirm:
1. The default agent is `plan` (analysis mode)
2. Subagents are available and correctly scoped

---

## Why "mnemosyne"
Mnemosyne is the Greek goddess of memory and the mother of the Muses. The name reflects the intent: a persistent configuration that shapes how each assistant recalls its role and routes work — consistently, across every session.

## License
MIT
