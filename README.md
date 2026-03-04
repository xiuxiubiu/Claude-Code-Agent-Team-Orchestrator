# Agent Teams

**A Claude Code skill for planning and coordinating native Agent Teams** — multi-specialist crews that communicate directly, share a task list, and execute in parallel.

> [!NOTE]
> **Breaking Change:** The command name has changed from `/agent-team-orchestrator` to `/agent-teams`. If you have the skill installed, please update your installation and use the new command.

> [!IMPORTANT]
> Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

---

## Table of Contents

- [What This Is](#what-this-is)
- [When to Use It](#when-to-use-it)
- [How It Works — The 4 Phases](#how-it-works--the-4-phases)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Repository Structure](#repository-structure)
- [Key Design Principles](#key-design-principles)
- [Contributing](#contributing)

---

## What This Is

This is a **Claude Code Skill** (invoked via `/agent-teams`) that acts as a team lead: it designs the right team composition, facilitates a collaborative planning session between specialists, resolves conflicts, and shepherds execution through a structured 4-phase process.

It uses Claude Code's **native Agent Teams** feature — where teammates run as independent agents, send peer-to-peer messages, and share a task list — not the standard `Agent` subagent tool.

---

## When to Use It

Use the orchestrator when your task benefits from **parallel independent work**, **specialized domain expertise**, or **adversarial review**:

| Scenario | Team Size | Planning Round? |
|----------|-----------|:---------------:|
| Multi-domain feature (backend + frontend + security) | 3–4 | Yes |
| Feature with unclear interface contracts | 3–4 | Yes |
| Security + code + performance review on same codebase | 2–3 | Yes |
| Decision requiring conflicting perspectives | 3 | Yes |
| Large refactoring with cross-module dependencies | 4–5 | Yes |
| Multi-domain feature with clear specs | 3–4 | No |

**Skip the orchestrator** for well-scoped, single-domain work. A single agent is faster and simpler.

> [!WARNING]
> **Token cost scales with team size.** Each teammate is an independent agent invocation with its own context window and API calls. A 3-person team can consume 3–5× the tokens of a single agent on the same task. Reserve the orchestrator for work where parallel execution or cross-domain expertise genuinely justifies the extra cost.

---

## How It Works — The 4 Phases

### Phase 1 — Role Assessment

The orchestrator maps the task's requirements to specialist roles. It applies a **minimum viable team** principle: every teammate must have non-overlapping deliverables. Teams of 3+ always include a challenger role (QA, devil's advocate, or security reviewer).

### Phase 2 — Approval Gate

A proposal table is presented listing each teammate, their `subagent_type`, and their key deliverables. No work begins until the user approves.

> [!NOTE]
> In Claude Code's Plan Mode, a plan file is written instead, which executes after approval.

### Phase 3 — Collaborative Planning *(when needed)*

Before any implementation begins, teammates exchange one round of peer-to-peer messages to surface dependencies, agree on interface contracts, and flag risks. The lead synthesizes input and resolves conflicts with final authority.

**Complexity gate** — Phase 3 runs if any of these are true:

- Teammates need to agree on API shapes or data structures
- Hidden dependencies are likely
- Roles have inherently conflicting priorities (e.g., security vs. performance)
- The work is novel, risky, or expensive to redo

### Phase 4 — Team Execution

Tasks are created with clear acceptance criteria and `blockedBy`/`blocks` dependency links. Teammates claim tasks, communicate only when something materially changes (blocked, task complete, unexpected discovery), and shut down gracefully when all work is done.

---

## Prerequisites

1. **Claude Code** installed (`npm install -g @anthropic-ai/claude-code`)
2. **Agent Teams enabled** — set in your environment or `~/.claude/settings.json`:

   ```bash
   # Environment variable
   export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
   ```

   ```jsonc
   // Or in ~/.claude/settings.json
   {
     "env": {
       "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
     }
   }
   ```

---

## Installation

Skills are installed manually by placing a `SKILL.md` file in the correct directory. Claude discovers them automatically — no restart needed.

### Method 1 — Clone *(recommended)*

Clones the full repo so the `references/` directory is available alongside `SKILL.md`:

```bash
git clone https://github.com/xiuxiubiu/agent-teams \
  ~/.claude/skills/agent-teams
```

### Method 2 — Copy `SKILL.md` only *(minimal)*

For users who only want the core protocol without the reference files:

```bash
mkdir -p ~/.claude/skills/agent-teams
curl -fsSL https://raw.githubusercontent.com/xiuxiubiu/agent-teams/main/SKILL.md \
  -o ~/.claude/skills/agent-teams/SKILL.md
```

### Method 3 — Project-scoped *(team sharing)*

Install into a project repo so all teammates get the skill automatically:

```bash
git clone https://github.com/xiuxiubiu/agent-teams \
  .claude/skills/agent-teams
```

Then commit `.claude/skills/` to version control.

### Verify and invoke

Type `/` in Claude Code — `agent-teams` should appear in autocomplete. Then describe your task and the orchestrator handles the rest:

```
/agent-teams
```

---

## Repository Structure

```
.
├── SKILL.md                        # Skill definition and orchestration instructions
└── references/
    ├── example.md                  # Complete worked example (REST API team)
    └── failure-scenarios.md        # Reference for 5 common failure modes
```

**`SKILL.md`** — The core skill file. Contains the full orchestration protocol: phase definitions, the teammate prompt template, the synthesis protocol, shutdown sequence, and quick-reference decision tables.

**`references/example.md`** — A complete worked example showing a 3-person REST API team (backend-dev, security-reviewer, doc-writer) walking through all four phases — role justification, approval, one planning exchange, lead synthesis, and task creation with dependencies.

**`references/failure-scenarios.md`** — Reference for 5 failure modes with root causes, resolution patterns, and anti-patterns:
1. Teammate rejects shutdown
2. Teammate unresponsive at shutdown
3. Task stalled with no owner
4. Circular task dependency
5. Messages not received (background agent bug)

---

## Key Design Principles

| Principle | Description |
|-----------|-------------|
| **Minimum viable team** | Every teammate must have unique, non-overlapping deliverables. Resist inflating team size — smaller teams communicate faster. |
| **Exception-driven communication** | Teammates message only when something materially changes: task complete, blocked, or unexpected discovery. No routine check-ins, FYI broadcasts, or seeking approval for decisions within one's own scope. |
| **Peer-to-peer negotiation** | Dependencies are resolved between teammates directly — the lead does not relay messages. This surfaces real constraints faster and reduces bottlenecks. |
| **Lead's decision is final** | Co-creation means input is solicited, not consensus required. The lead resolves conflicts and assigns tasks after one planning round. |
| **No background agents** | Teammates must never be spawned with `run_in_background: true`. Background agents cannot register with the message bus and will be invisible to the team. |

---

## Contributing

See [CONTRIBUTING.md](.github/CONTRIBUTING.md) for guidelines.

Bug reports and feature suggestions: [open an issue](https://github.com/xiuxiubiu/agent-teams/issues).

## License

MIT
