# Agent Team Orchestrator

**A Claude Code skill for planning and coordinating native Agent Teams** — multi-specialist crews that communicate directly, share a task list, and execute in parallel.

> Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

---

## What This Is

This is a **Claude Code Skill** (invoked via `/agent-team-orchestrator`) that acts as a team lead: it designs the right team composition, facilitates a collaborative planning session between specialists, resolves conflicts, and shepherds execution through a structured 4-phase process.

It uses Claude Code's **native Agent Teams** feature — where teammates run as independent agents, send peer-to-peer messages, and share a task list — not the standard `Agent` subagent tool.

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

## How It Works — The 4 Phases

### Phase 1 — Role Assessment
The orchestrator maps the task's requirements to specialist roles. It applies a **minimum viable team** principle: every teammate must have non-overlapping deliverables. Teams of 3+ always include a challenger role (QA, devil's advocate, or security reviewer).

### Phase 2 — Approval Gate
A proposal table is presented listing each teammate, their `subagent_type`, and their key deliverables. No work begins until the user approves.

*(In Claude Code's Plan Mode, a plan file is written instead, which executes after approval.)*

### Phase 3 — Collaborative Planning (when needed)
Before any implementation begins, teammates exchange one round of peer-to-peer messages to surface dependencies, agree on interface contracts, and flag risks. The lead synthesizes input and resolves conflicts with final authority.

**Complexity gate** — Phase 3 runs if any of these are true:
- Teammates need to agree on API shapes or data structures
- Hidden dependencies are likely
- Roles have inherently conflicting priorities (e.g., security vs. performance)
- The work is novel, risky, or expensive to redo

### Phase 4 — Team Execution
Tasks are created with clear acceptance criteria and `blockedBy`/`blocks` dependency links. Teammates claim tasks, communicate only when something materially changes (blocked, task complete, unexpected discovery), and shut down gracefully when all work is done.

## Prerequisites

1. **Claude Code** installed (`npm install -g @anthropic-ai/claude-code`)
2. **Agent Teams enabled** — set in your environment or `~/.claude/settings.json`:

   ```bash
   # Environment variable
   export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1

   # Or in ~/.claude/settings.json
   {
     "env": {
       "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
     }
   }
   ```

## Installation

Install via the Claude Code skill system. Add the skill to your `~/.claude/skills/` directory or use a skill registry that supports it.

Once installed, invoke with:

```
/agent-team-orchestrator
```

Then describe your task. The orchestrator handles the rest.

## Repository Structure

```
.
├── SKILL.md                        # Skill definition and orchestration instructions
└── references/
    ├── example.md                  # Complete worked example (REST API team)
    └── failure-scenarios.md        # Reference for 5 common failure modes
```

### `SKILL.md`
The core skill file. Contains the full orchestration protocol: phase definitions, the teammate prompt template, the synthesis protocol, shutdown sequence, and quick-reference decision tables.

### `references/example.md`
A complete worked example showing a 3-person REST API team (backend-dev, security-reviewer, doc-writer) walking through all four phases — role justification, approval, one planning exchange, lead synthesis, and task creation with dependencies.

### `references/failure-scenarios.md`
Reference for 5 failure modes with root causes, resolution patterns, and anti-patterns:
1. Teammate rejects shutdown
2. Teammate unresponsive at shutdown
3. Task stalled with no owner
4. Circular task dependency
5. Messages not received (background agent bug)

## Key Design Principles

**Minimum viable team.** Every teammate must have unique, non-overlapping deliverables. Resist inflating team size — smaller teams communicate faster.

**Exception-driven communication.** Teammates message only when something materially changes: task complete, blocked, or unexpected discovery. No routine check-ins, FYI broadcasts, or seeking approval for decisions within one's own deliverables.

**Peer-to-peer negotiation.** Dependencies are resolved between teammates directly — the lead does not relay messages. This surfaces real constraints faster and reduces bottlenecks.

**Lead's decision is final.** Co-creation means input is solicited, not consensus required. The lead resolves conflicts and assigns tasks after one planning round.

**No background agents.** Agent Team teammates must never be spawned with `run_in_background: true`. Background agents cannot register with the message bus and will be invisible to the team.

## Contributing

See [CONTRIBUTING.md](.github/CONTRIBUTING.md) for guidelines.

Bug reports and feature suggestions: [open an issue](https://github.com/xiuxiubiu/Claude-Code-Agent-Team-Orchestrator/issues).

## License

MIT
