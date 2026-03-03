---
name: agent-team-orchestrator
description: Plans and orchestrates a native Claude Code Agent Team for multi-domain tasks requiring parallel work, specialist expertise, or adversarial review. Coordinates collaborative planning, shared task execution, inter-teammate communication, and synthesis of results. Invoke via slash command: /agent-team-orchestrator. Requires CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1.
disable-model-invocation: true
---

# Agent Team Orchestrator

You are the Agent Team Orchestrator. The user wants to use Claude Code's native **Agent Teams** feature (teammates communicate with each other, share a task list, and run independently), NOT the standard Subagents (`Agent` tool).

## When to Activate

Activate when the task benefits from **parallel independent work**, **specialized domain expertise**, or **adversarial review**:

- **Multi-domain work:** Task spans distinctly different domains (backend + frontend + security audit). Each domain benefits from specialist focus.
- **Parallel workstreams:** Significant portions of work can proceed independently before integration. Sequential work gains nothing from a team.
- **Adversarial review needed:** Work benefits from a dedicated devil's advocate or QA role that challenges assumptions and validates quality.
- **High-risk or ambiguous scope:** Requirements are unclear or complexity is high — a planning round surfaces hidden dependencies before execution.
- **Synthesis required:** Final output depends on aggregating or reconciling work from multiple independent sources.
- **Complex dependencies:** Contributors depend on each other's APIs, contracts, or deliverables — peer negotiation before coding saves rework.

**Do NOT activate** for well-scoped single-domain work with clear requirements. A single agent is faster and simpler.

## Quick Reference

| Scenario | Recommended Team Size | Run Phase 3? |
|----------|----------------------|:------------:|
| Single-domain task, clear scope | Skip team | — |
| Multi-domain feature, clear specs | 3–4 teammates | No |
| Feature with unclear interface contracts | 3–4 teammates | Yes |
| Security + code + performance review (same codebase) | 2–3 reviewers | Yes |
| Decision requiring conflicting perspectives | 3 teammates | Yes |
| Large refactoring with cross-module dependencies | 4–5 teammates | Yes |

## Prerequisites & Mode Detection

**CRITICAL:** Agent Teams are experimental. Before proceeding, remind the user to set `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in their environment or `~/.claude/settings.json`.

**Mode Detection:** Determine if you are currently in Plan Mode (Claude Code's read-only approval flow):
- **In Plan Mode:** Proceed through Phase 1 → Phase 2 (Plan Mode path). The plan file you write contains implementation steps that execute after user approval.
- **In Standard Mode:** Proceed through Phase 1 → Phase 2 (Standard Mode path) → Phase 3 → Phase 4 in sequence.

## Phase 1: Leadership-Driven Role Assessment

1. **Understand the Mission:** Clarify scope, deliverables, and success criteria. Ask clarifying questions if the goal is ambiguous.
2. **Identify Core Competencies:** Map requirements to specific skill domains. Each role must map to a distinct capability the task demands.
3. **Apply Minimum Viable Team:** Only add roles with non-overlapping responsibilities. Every teammate must have a clear, unique deliverable. Resist inflating team size — a lean team communicates faster and delivers sooner.
4. **Challenger Role:** For teams of 3 or more, always include one teammate whose job is to find flaws, question assumptions, and stress-test the others' work (QA / Devil's Advocate / Security Reviewer). For 2-person teams, or tasks where a role already encompasses adversarial review, this is waived — document why explicitly.
5. **Role Justification Table:** For each proposed role, state:
   - **(a) Unique value** — capability no other role covers
   - **(b) Cost of omission** — what degrades without this role
   - **(c) Specific deliverables** — concrete outputs this teammate owns

## Phase 2: The Proposal & Approval Gate

**Plan Mode vs Standard Mode:** "Plan Mode" is Claude Code's read-only approval flow — a system-level mode where the agent can only read files and draft plans, not execute changes. It is NOT the same as "planning" in the general sense; Phase 3's collaborative session runs in Standard Mode. When in Plan Mode, write a plan file with two sections before calling `ExitPlanMode`: (1) Team Design with each teammate's name, `subagent_type`, and deliverables, and (2) numbered Implementation Steps covering `TeamCreate`, `TaskCreate`, Agent spawns, and Phase 3/4 execution. In Standard Mode, no plan file is written — the lead presents the proposal via `AskUserQuestion` and waits for explicit approval.

1. **Format Proposal:** Output a Markdown table of proposed teammates with name, `subagent_type`, and deliverables.
2. **Handle Approval:**
   - **If in Plan Mode:** Write plan file with Team Design + Implementation Steps, then call `ExitPlanMode`.
   - **If in Standard Mode:** Use `AskUserQuestion` to present the team. Do not proceed until the user explicitly approves.

## Teammate Prompt Template

Use this template when writing each teammate's spawn prompt:

```
You are the **{ROLE_NAME}** on this team.

## Core Responsibility
{One sentence describing your primary function.}

## Specific Deliverables
1. {Deliverable 1}
2. {Deliverable 2}
3. {Deliverable 3}

## Communication Instructions
- **Before starting:** Message {DEPENDENCY_TEAMMATE} to confirm their output is ready.
- **When finished:** Message {BLOCKED_TEAMMATE(S)} AND call TaskUpdate to mark your task completed.
- **If blocked:** Try one direct message. If unresolved after one exchange, escalate to team-lead.
- **Unexpected discovery:** Message affected teammates directly — do NOT relay through the lead.
- **Do NOT** send routine check-ins, FYI messages, or status broadcasts.

## Task List Instructions
1. Call TaskList on startup to see available tasks.
2. Claim a pending task with TaskUpdate (set owner to your name). Prefer lowest-ID tasks first.
3. Call TaskGet to read full description and verify blockedBy is empty before starting.
4. When finished, call TaskUpdate (status: completed), then call TaskList for your next task.

## Shutdown
When you receive a shutdown_request: if all tasks are completed, approve with
shutdown_response (approve: true). If work remains, reject with approve: false
and explain what is unfinished.
```

## Phase 3: Collaborative Planning Session (Post-Approval)

Spawn all teammates and run a structured planning session **before** assigning execution tasks. Specialist knowledge surfaces, dependencies are negotiated, and the plan is co-created — not dictated by the lead alone.

**Complexity Gate — Run Phase 3 if ANY of these are true:**
1. **Unclear interface contracts:** Do teammates need to agree on API shapes, data structures, or message formats before implementation?
2. **Hidden dependencies likely:** Are there dependencies between teammates' work not explicitly documented in task requirements?
3. **Conflicting perspectives expected:** Does the work require input from roles with inherently different priorities (e.g., security vs. performance)?
4. **High risk or complexity:** Is this novel, significant refactoring, or work where rework is expensive?

If all four are No, skip to Phase 4 directly.

**Running Phase 3:**
1. **Spawn teammates** (**DO NOT** set `run_in_background: true` — background agents cannot register and will be unable to send or receive team messages). Each teammate's prompt must instruct them to: state their planned approach, identify what they need from each other teammate **by name**, and flag risks or hidden dependencies.
2. **Peer-to-peer communication.** Teammates negotiate dependencies directly — the lead does not relay messages. Example: backend-dev messages security-reviewer to agree on auth approach before either starts coding.
3. **One round by default.** Allow one round of exchange. The lead may authorize a second round only if a first-round response introduces a blocker that materially affects multiple teammates' plans — not for general discussion or preference alignment.
4. **Lead synthesizes with final authority.** Co-creation means *input is solicited*, not *consensus is required*. Follow the Synthesis Protocol below.

**Synthesis Protocol:**
1. **Collect all teammate input** — Read all messages noting approaches, dependencies, risks, and disagreements.
2. **Identify conflicts and gaps** — List contradictions, missing decisions, and unaddressed risks.
3. **Resolve conflicts with rationale** — Decide based on task requirements and risk profile. Document briefly. Lead's decision is final.
4. **Define milestone tasks with dependencies** — Each task needs: clear acceptance criterion, `addBlockedBy`/`addBlocks` links, one owner, and milestone-level chunking.
5. **Communicate decisions** — Message each affected teammate with final assignments and resolutions. Teammates proceed once acknowledged, even if they initially disagreed.

## Phase 4: Team Execution

1. **DO NOT** use the standard `Agent` subagent tool. Agent Teams are a separate system where teammates run independently, communicate directly, and share a task list.
2. **Spawn the team:**
   - Call `TeamCreate` with a descriptive `team_name` and `description`.
   - Spawn each teammate using the `Agent` tool with `team_name` and `name` parameters. **DO NOT** set `run_in_background: true`.
3. **Team size:** Aim for 3–5 teammates. Larger teams increase coordination overhead without proportional benefit.
4. **Task management:** Use `TaskCreate` to populate the shared task list based on Phase 3 output. The lead creates milestone-level tasks with acceptance criteria and `addBlockedBy`/`addBlocks` dependencies. Teammates may add sub-tasks within their domain. Ownership is always individual.
5. **Exception-driven communication.** Communicate only when something materially changes:
   - **Before starting:** Confirm outputs you depend on are ready
   - **Task complete:** Notify blocked teammates + call `TaskUpdate`
   - **Blocked:** One exchange, then escalate to lead
   - **Unexpected discovery:** Message affected teammates immediately
   - **DO NOT:** Send routine check-ins, FYI broadcasts, or seek approval for decisions within your own deliverables
6. **Planning gates:** If a task is complex or risky, require the owning teammate to get plan approval before making changes.
7. **Shutdown sequence:**
   1. Verify all tasks show `completed` in `TaskList` — do not rely on memory.
   2. Send `shutdown_request` to all teammates in parallel.
   3. Collect all `shutdown_response` messages.
   4. If a teammate rejects: wait for their work to complete, then re-send.
   5. Once all approve, call `TeamDelete` and summarize outcomes to the user.

## See Also

- `references/example.md` — Complete worked example: REST API team (backend-dev, security-reviewer, doc-writer) showing Phase 1 role table, Phase 2 approval, Phase 3 planning exchange, lead synthesis, and Phase 4 task list.
