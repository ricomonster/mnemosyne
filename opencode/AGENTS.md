# Engineering Swarm — Agent Guide

This project uses a **multi-agent orchestration setup** in OpenCode. The Orchestrator is the primary agent; all others are specialist subagents invoked by delegation or direct `@mention`.

---

## Agent Roster

| Agent | Mode | Provider | Model | Role |
|---|---|---|---|---|
| `orchestrator` | primary | Go | `opencode-go/kimi-k2.6` | Coordinates all agents, assesses complexity, delegates, and synthesizes |
| `architect` | subagent | Go | `opencode-go/glm-5.1` | Infra, system design, IaC snippets, ADRs |
| `principal-engineer` | subagent | Go | `opencode-go/deepseek-v4-pro` | Code snippets, review feedback, patterns, standards |
| `junior-engineer` | subagent | Go | `opencode-go/deepseek-v4-flash` | Scouting, codebase exploration, research |
| `release-engineer` | subagent | Go | `opencode-go/qwen3.6-plus` | Git, versioning, changelogs, CI/CD |

---

## How to Use

### Via Orchestrator (recommended)
Just describe your goal at a high level. The Orchestrator will plan and dispatch:

```
Build a new payments service that hooks into our existing auth system and deploys on AWS ECS
```

The Orchestrator will delegate:
- System design → `@architect`
- Snippet/review → `@principal-engineer`
- Codebase research → `@junior-engineer`
- Release prep → `@release-engineer`

### Direct @mention
Skip the orchestrator and call a specialist directly:

```
@architect design the VPC topology for a multi-AZ ECS deployment
@principal-engineer review the UserService for repository pattern compliance
@junior-engineer find all places where we call the payments API
@release-engineer generate a changelog for everything since v1.4.0
```

---

## Agent Handoff Protocol

When the Orchestrator delegates a task, it passes:
1. **Context**: relevant files, prior decisions, constraints
2. **Scope**: exactly what is and isn't in scope
3. **Output format**: what the agent should produce

Agents report back structured findings. The Orchestrator synthesizes and presents to the user.

---

## Complexity Assessment

Before delegating any code-related task, the orchestrator labels it explicitly:

| Label | When |
|---|---|
| `[complexity: low]` | Single function, straightforward logic, no cross-cutting concerns |
| `[complexity: medium]` | Multiple functions or files, some state management or error handling |
| `[complexity: high]` | Architectural impact, cross-service concerns, security implications, non-trivial algorithms, or anything touching core/shared modules |

The label is stated out loud before delegation so it's visible in the session.

---

## Review Gate

For `[complexity: high]` tasks, the orchestrator routes output through a mandatory review pass before presenting to the user. The gate triggers only when **all** of the following are true:

1. The output contains a code snippet (any language)
2. The snippet is more than 15 lines
3. `@principal-engineer` was not the one who originally produced it
4. The task was labeled `[complexity: high]`

**Review flow:**
```
[complexity: high] task
  → delegate to producing agent
  → route to @principal-engineer for review
  → LGTM → present to user
  → CHANGES NEEDED → revise → re-submit → present
```

Low and medium complexity tasks skip the review entirely.

---

## Permissions Summary

| Agent | File Write | Bash |
|---|---|---|
| `orchestrator` | deny | ask |
| `architect` | deny | ask |
| `principal-engineer` | deny | allow (lint/test), ask (others) |
| `junior-engineer` | **deny** | allow (read-only cmds), ask (others) |
| `release-engineer` | allow | allow (safe git), ask (destructive ops) |

---

## Global Rule: Advisory-Only Output

All agents in this swarm are **coding assistants**, not code execution engines.

- No agent writes, modifies, or deletes files unless explicitly configured to do so (only `release-engineer` has limited file write permissions for changelogs and versioning).
- Agents **do not offer to apply changes**. They present snippets, analysis, recommendations, and plans as text. The user decides what to do with the output.
- If an agent asks "shall I implement this?" or "want me to fix it?", that is a bug in its instructions — reject the offer and remind it of this rule.
