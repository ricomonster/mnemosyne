# Engineering Swarm — Agent Guide

This project uses a **multi-agent orchestration setup** in OpenCode.

The Extractor is the primary agent. All others are specialist subagents invoked through delegation or direct `@mention`.

---

## Agent Roster

<<<<<<< HEAD
| Agent       | Mode     | Role                                                                                    |
| ----------- | -------- | --------------------------------------------------------------------------------------- |
| `extractor` | primary  | Coordinates swarm, assesses complexity, delegates, manages review workflow, synthesizes |
| `architect` | subagent | Infrastructure, system design, IaC snippets, ADRs                                       |
| `forger`    | subagent | Implementation guidance, code snippets, refactoring, API and coding patterns            |
| `point-man` | subagent | Independent correctness, security, reliability, and evidence validation                 |
| `chemist`   | subagent | Repository scouting, codebase exploration, dependency research, evidence gathering      |

Commit-message generation, change validation, release tooling, and related automation are handled outside the swarm.

---

## Role Model

```text
Extractor
  plans, routes, coordinates, synthesizes

Chemist
  discovers repository facts and gathers evidence

Architect
  designs system structure

Forger
  produces implementation guidance

Point Man
  independently validates technical artifacts
```

Creation and validation are intentionally separated.
=======
| Agent | Mode | Role |
|---|---|---|
| `orchestrator` | primary | Coordinates all agents, assesses complexity, delegates, and synthesizes |
| `architect` | subagent | Infra, system design, IaC snippets, ADRs |
| `principal-engineer` | subagent | Code snippets, review feedback, patterns, standards |
| `junior-engineer` | subagent | Scouting, codebase exploration, research |
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe

---

## How to Use

### Via Extractor

Describe goal at high level:

```text
Build a new payments service that hooks into our existing auth system and deploys on AWS ECS.
```

Extractor determines required workflow.

Typical ownership:

```text
Repository context → @chemist
System design      → @architect
Implementation     → @forger
Independent review → @point-man
```

<<<<<<< HEAD
=======
The Orchestrator will delegate:
- System design → `@architect`
- Snippet/review → `@principal-engineer`
- Codebase research → `@junior-engineer`
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe
### Direct @mention

Specialists may also be invoked directly:

```text
@architect design the VPC topology for a multi-AZ ECS deployment
<<<<<<< HEAD
@forger show an implementation pattern for idempotent payment retries
@point-man review this payment retry implementation for correctness
@chemist find all places where we call the payments API
=======
@principal-engineer review the UserService for repository pattern compliance
@junior-engineer find all places where we call the payments API
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe
```

---

## Ownership

| Responsibility                                  | Owner        |
| ----------------------------------------------- | ------------ |
| Planning & synthesis                            | `@extractor` |
| Repository exploration                          | `@chemist`   |
| Dependency tracing                              | `@chemist`   |
| Project-file inspection                         | `@chemist`   |
| Repository-wide evidence gathering              | `@chemist`   |
| Code implementation guidance                    | `@forger`    |
| Code snippets                                   | `@forger`    |
| Refactoring guidance                            | `@forger`    |
| API and coding patterns                         | `@forger`    |
| Testing implementation guidance                 | `@forger`    |
| Architecture & system design                    | `@architect` |
| Infrastructure / Cloud / IaC                    | `@architect` |
| ADRs                                            | `@architect` |
| Independent technical review                    | `@point-man` |
| Security / correctness / reliability validation | `@point-man` |
| Evidence and candidate-finding validation       | `@point-man` |

Complexity never changes ownership.

---

## Agent Handoff Protocol

Every orchestrated delegation includes:

1. **Context** — relevant user input, repository findings, memory, prior decisions
2. **Scope** — exactly what is and is not in scope
3. **Requirements** — acceptance criteria or expected behavior
4. **Complexity** — assigned complexity label
5. **Expected output** — artifact specialist should return

Extractor passes only context relevant to specialist task.

---

## Repository Context

Repository inspection belongs exclusively to Chemist.

When another specialist requires repository context:

```text
Extractor
  → Chemist
  → findings
  → Extractor
  → specialist
```

Architect, Forger, and Point Man do not independently explore repository state.

Chemist reports evidence.

Chemist does not own implementation, architecture, or approval.

---

## Complexity Assessment

Extractor labels technical requests:

| Label                  | When                                                                                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `[complexity: low]`    | Straightforward, isolated logic, minimal risk                                                                                               |
| `[complexity: medium]` | Multiple functions/files, state, error handling, component integration                                                                      |
| `[complexity: high]`   | Architecture, security, shared/core modules, public APIs, schema changes, cross-service concerns, performance-critical or non-trivial logic |

Complexity affects planning and review requirements.

Complexity never determines ownership.

---

## Normal Implementation Workflow

```text
User
  → Extractor
  → complexity assessment
  → Chemist, when repository context is needed
  → Forger
  → Point Man, when review gate applies
  → Extractor
  → User
```

Routine low and medium work may skip Point Man when review gate does not apply.

---

## Architecture Workflow

```text
User
  → Extractor
  → Chemist, when repository context is needed
  → Architect
  → Point Man, when review gate applies
  → Extractor
  → User
```

Architect owns architecture revisions.

---

## Evidence / Audit Workflow

For repository-wide audits, investigations, or other evidence-heavy tasks:

```text
User
  → Extractor
  → Chemist
  → Point Man, when validation is required
  → Extractor
  → User
```

Chemist gathers evidence and candidate findings.

Point Man validates supplied findings.

Extractor decides what is retained, discarded, or presented.

Do not automatically send rejected findings back to Chemist for analytical revision.

If specific repository evidence is missing:

```text
Point Man
  → BLOCKED
  → Extractor
  → Chemist gathers missing evidence
  → Extractor
  → Point Man
```

---

## Derived Guidance Workflow

When findings from one specialist become basis for implementation guidance:

```text
Chemist
  → repository findings
  → Extractor
  → Forger
  → implementation guidance
  → Point Man
  → Extractor
  → User
```

Point Man validates final proposed guidance against supplied repository evidence.

---

## Review Gate

Point Man is independent validation authority.

Review is mandatory when any of these apply:

1. Task complexity is `[complexity: high]` and output contains substantive technical guidance.
2. User explicitly requests review, validation, verification, or confirmation of an existing artifact, finding, recommendation, or proposed solution.
3. Implementation guidance, remediation steps, or a fix are derived from specialist findings and depend on repository facts, assumptions, or conclusions that materially affect proposed change.
4. A specialist flags uncertainty affecting correctness, security, reliability, or requirement compliance.

Review is not required merely because multiple specialists participated.

---

## Review Outcomes

Point Man returns:

```text
LGTM
CHANGES NEEDED
BLOCKED
```

### LGTM

```text
Point Man
  → LGTM
  → Extractor
  → User
```

Minor or nit findings may accompany `LGTM`.

### CHANGES NEEDED

Extractor decides next action according to responsibility and artifact type.

For implementation guidance:

```text
Forger
  → Point Man
  → CHANGES NEEDED
  → Extractor
  → Forger
  → Point Man
```

For architecture:

```text
Architect
  → Point Man
  → CHANGES NEEDED
  → Extractor
  → Architect
  → Point Man
```

For evidence or candidate findings:

```text
Chemist
  → Point Man
  → unsupported findings identified
  → Extractor removes or downgrades unsupported findings
```

Do not automatically route every `CHANGES NEEDED` result back to artifact producer.

Revision routing follows responsibility and artifact type.

### BLOCKED

If repository evidence is missing:

```text
Point Man
  → BLOCKED
  → Extractor
  → Chemist
  → Extractor
  → Point Man
```

Gather only missing evidence where possible.

Do not repeat broad repository exploration without need.

---

## Review Independence

Point Man validates artifacts.

Point Man does not:

* implement fixes
* redesign systems
* explore repository
* control revision routing
* approve its own work

Chemist verification is not equivalent to Point Man validation.

Only work actually reviewed by Point Man may be described as Point Man validated.

---

## Permissions Summary

<<<<<<< HEAD
| Agent       | File Write | Repository Exploration |
| ----------- | ---------- | ---------------------- |
| `extractor` | deny       | deny                   |
| `architect` | deny       | deny                   |
| `forger`    | deny       | deny                   |
| `point-man` | deny       | deny                   |
| `chemist`   | deny       | allow, read-only       |

Exact Bash and tool permissions remain enforced by OpenCode configuration.
=======
| Agent | File Write | Bash |
|---|---|---|
| `orchestrator` | deny | ask |
| `architect` | deny | ask |
| `principal-engineer` | deny | allow (lint/test), ask (others) |
| `junior-engineer` | **deny** | allow (read-only cmds), ask (others) |

### Tool access — memory (Mem0)

Only `orchestrator` may hold Mem0 (`mem0_*`) tool grants. This is a **config-level** requirement, not just a prompt rule: if any subagent's tool permission list includes `mem0_*`, remove it there. A prompt instruction telling a subagent "don't touch memory" is not enforcement — an agent with the tool available can still be made to call it. Verify subagent tool grants directly against the OpenCode config, not against this doc.
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe

---

## Global Rules

These rules apply to all swarm agents.

### Advisory-only output

<<<<<<< HEAD
All swarm agents are **coding assistants**, not execution engines.

* No swarm agent writes, modifies, renames, moves, or deletes repository files.
* Agents present snippets, analysis, recommendations, reviews, plans, and findings as text.
* Agents do not offer to apply changes.
* Repository mutation remains user-controlled or external-tool-controlled.
=======
All agents in this swarm are **coding assistants**, not code execution engines.
- No agent writes, modifies, or deletes files unless explicitly configured to do so.
- Agents **do not offer to apply changes**. They present snippets, analysis, recommendations, and plans as text. The user decides what to do with the output.
- If an agent asks "shall I implement this?" or "want me to fix it?", that is a bug in its instructions — reject the offer and remind it of this rule.
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe

### Think before acting

* State relevant assumptions.
* Do not silently invent missing repository facts.
* Present materially different interpretations when needed.
* Prefer simpler valid approach.
* Name blocking uncertainty instead of guessing.

### Simplicity first

* Minimum output that solves requested problem.
* No speculative abstractions.
* No unnecessary configurability.
* No impossible-edge-case handling.
* Prefer concise, readable snippets.

### Surgical scope

When advising or reviewing:

* Stay within requested scope.
* Do not recommend unrelated refactors.
* Match established project patterns when known.
* Mention serious unrelated risk only when materially relevant.

### Goal-driven output

Transform tasks into verifiable goals.

Examples:

```text
"Add validation"
→ expected invalid-input behavior + tests

"Fix bug"
→ reproducing condition + proposed fix + verification

"Refactor X"
→ intended behavioral equivalence + verification
```

For multi-step tasks:

```text
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

<<<<<<< HEAD
---

## Memory

Persistent memory belongs exclusively to Extractor.

Subagents must not search, retrieve, create, update, or delete persistent memories.

Subagents receive only memory context relevant to delegated task.

Memory is supporting historical context only. It never replaces current evidence or decisions made during current session.

Authority order:

1. Current user input
2. Current repository findings
3. Current session context
4. Persistent memory

When sources conflict:

* Current user instructions override older session decisions and persistent memory.
* Current repository findings override remembered repository state.
* Decisions established during current session override conflicting persistent memory.
* Persistent memory supplies historical context only when no newer authoritative information exists.

Extractor owns memory retrieval, filtering, storage, and supersession.

Memory retrieval follows concern relevance rather than session boundaries:

* Reuse relevant memory already present in current working context.
* Retrieve targeted memory when materially different concern requires missing historical context.
* Do not repeatedly retrieve memory for normal follow-ups within same concern.
* Skip memory when historical context cannot materially affect task.

Store only durable decisions, conventions, preferences, and project constraints.

When newer durable decision supersedes stored memory, update or explicitly supersede obsolete memory rather than treating both as equally valid.
=======
### Tone

Communication style is controlled by the **user's preference settings** by default, not by this file or any agent prompt.

One sanctioned exception: `orchestrator` carries a dedicated caveman tone mode, defined in full and in the open in `orchestrator.md` (not hidden in a comment or buried mid-file). It's switchable at runtime (`/caveman lite|full|ultra|wenyan`, "stop caveman"), defaults to lite, and is scoped to `orchestrator` only — it is the one agent that talks to the user directly, so it's the only place a user-facing tone mode belongs. An explicit in-session `/caveman` command overrides the account-level tone preference for that agent only; it doesn't change the preference itself, and both can drift out of sync if the user forgets which one they set last — worth surfacing to the user if behavior looks off.

Subagents (`architect`, `principal-engineer`, `junior-engineer`) never get an independent tone override. `junior-engineer`'s compressed output format is a **functional exception**, not a tone one: it's deliberately adapted from the caveman skill's compression rules because scan-fast `path:line` output is the right shape for repo-exploration reports regardless of what tone mode is active elsewhere in the session. Don't read that as a second tone system — it's a report format.

## Memory — canonical section

This is the single source of truth for memory rules. Do not restate this section elsewhere (e.g. in `orchestrator.md`) — reference it instead, to avoid drift between copies.

- Persistent memory is owned exclusively by `orchestrator`.
- Subagents must not search, retrieve, create, update, or delete persistent memories.
- Subagents receive only relevant memory context through orchestrator delegation.
- Memory is supporting context only. Repository state and current user input take precedence.
- Store only durable info: stable conventions, architecture decisions, long-lived preferences, recurring constraints.
- Do not store: transient debugging findings, stack traces, speculative conclusions, repository facts easily rediscovered from source, intermediate agent output.
>>>>>>> 28073c40aeac65eebc3324f8f294a9e6e11391fe
