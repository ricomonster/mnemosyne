# mnemosyne

A Claude Code project configuration that enforces a structured, agent-based workflow with a strict analysis-only operating mode.

## What it is

Mnemosyne is a `.claude/` configuration layer that gives Claude Code a defined set of roles, boundaries, and escalation rules. Instead of Claude acting as a generalist assistant, it routes tasks to specialized agents — each with a fixed model, a narrow scope, and explicit limits on what it can do.

The default mode is **read-only analysis**: Claude researches, reasons, and advises. All execution (file changes, git operations) is performed explicitly by the user or triggered through the release-engineer agent.

---

## Structure

```
claude/
  ├── CLAUDE.md              # Global instructions: operating mode, principles, agent routing
  └── agents/
  ├── junior-engineer.md
  ├── senior-engineer.md
  ├── architect.md
  ├── principal-engineer.md
  └── release-engineer.md
```

---

## Operating mode

- Analysis only by default: no file creation, modification, or deletion
- No unsolicited offers to implement ("shall I...", "should I...")
- All mutations are user-initiated or go through `release-engineer`
- Bash access is read-only: `git log/diff/status`, `grep`, test runners

---

## Agents

| Agent | Model | Role |
|---|---|---|
| `junior-engineer` | Haiku | Repository exploration: find, list, grep, trace symbols |
| `senior-engineer` | Sonnet | Implementation, debugging, refactoring, code review |
| `architect` | Opus | System design, API contracts, tradeoff analysis, pre-implementation planning |
| `principal-engineer` | Opus | System-wide risk, scalability, long-term architectural validation |
| `release-engineer` | Sonnet | Git operations: stage, commit, push — with safety checks |

### Boundaries

**junior-engineer** — observes only. No quality judgments, no suggestions, no design inference.

**senior-engineer** — builds and fixes. Does not decide service boundaries or system architecture.

**architect** — designs before code is written. Does not write production code or debug.

**principal-engineer** — evaluates risk at the system level. Not part of the default workflow. Only invoked when genuine system-wide concerns are
  present.

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
- Claude Max or API access with model permissions for Haiku, Sonnet, and Opus

### Installation

Copy the contents into `~/.claude/`:

```sh
mkdir -p ~/.claude/agents
cp claude/CLAUDE.md ~/.claude/CLAUDE.md
cp claude/settings.json ~/.claude/settings.json
cp claude/agents/* ~/.claude/agents/
```

Claude Code loads `.claude/CLAUDE.md` and `.claude/settings.json` automatically when present.

### What gets installed

- **CLAUDE.md**: Operating mode, agent routing rules, and workflow definitions
- **settings.json**: Permission rules (deny `Edit`/`Write`, allow read-only operations)
- **agents/**: Individual agent definition files with model assignments

### Verify

Open Claude Code in any project and confirm:

1. Analysis-only mode is active (file edit operations require explicit approval)
2. Agent list appears with `/agents` command

You should see `junior-engineer`, `senior-engineer`, `architect`, `principal-engineer`, and `release-engineer`.

---

## Why "mnemosyne"

Mnemosyne is the Greek goddess of memory and the mother of the Muses. The name reflects the intent: a persistent configuration that shapes how Claude recalls its role and routes work — consistently, across every session.
