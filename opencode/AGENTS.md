# Engineering Swarm — Agent Guide

This project uses a **multi-agent orchestration setup** in OpenCode. The Orchestrator is the primary agent; all others are specialist subagents invoked by delegation or direct `@mention`.

---

## Agent Roster

| Agent | Mode | Model | Role |
|---|---|---|---|
| `orchestrator` | primary | `opencode-go/kimi-k2.6` | Coordinates all agents, plans and delegates |
| `architect` | subagent | `opencode-go/glm-5.1` | Infra, system design, IaC, ADRs |
| `principal-engineer` | subagent | `opencode-go/deepseek-v4-pro` | Code implementation, patterns, standards |
| `junior-engineer` | subagent | `opencode-go/deepseek-v4-flash` | Scouting, codebase exploration, research |
| `release-engineer` | subagent | `opencode-go/qwen3.6-plus`| Git, versioning, changelogs, CI/CD |

---

## How to Use

### Via Orchestrator (recommended)
Just describe your goal at a high level. The Orchestrator will plan and dispatch:

```
Build a new payments service that hooks into our existing auth system and deploys on AWS ECS
```

The Orchestrator will delegate:
- System design → `@architect`
- Implementation → `@principal-engineer`
- Codebase research → `@junior-engineer`
- Release prep → `@release-engineer`

### Direct @mention
Skip the orchestrator and call a specialist directly:

```
@architect design the VPC topology for a multi-AZ ECS deployment

@principal-engineer refactor the UserService to use the repository pattern

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

## Permissions Summary

| Agent | File Write | Bash |
|---|---|---|
| `orchestrator` | deny | ask |
| `architect` | ask | ask |
| `principal-engineer` | deny | allow (build/test/lint), ask (others) |
| `junior-engineer` | **deny** | allow (read-only cmds), ask (others) |
| `release-engineer` | allow | allow (safe git), ask (destructive ops) |

## Global Rule: Advisory-Only Output

All agents in this swarm are **coding assistants**, not code execution engines.

- No agent writes, modifies, or deletes files unless explicitly configured to do so (only `release-engineer` has limited file write permissions for changelogs and versioning).
- Agents **do not offer to apply changes**. They present snippets, analysis, recommendations, and plans as text. The user decides what to do with the output.
- If an agent asks "shall I implement this?" or "want me to fix it?", that is a bug in its instructions — reject the offer and remind it of this rule.
