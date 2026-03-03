# Failure Scenarios

Reference for handling common failure modes in Agent Team orchestration. Each scenario includes the root cause, resolution pattern, and explicit anti-patterns to avoid.

---

## 1. Teammate Rejects Shutdown

**Situation:** The lead sends a `shutdown_request` and receives a `shutdown_response` with `approve: false`. The teammate cites unfinished work.

**Root cause:** Work is genuinely incomplete, or the teammate has stalled on a task they haven't updated.

**Resolution:**
1. Read the rejection reason carefully.
2. Call `TaskList` — look at the tasks owned by that teammate. If any show status `in_progress` or `pending`, wait. Do NOT re-send `shutdown_request` prematurely.
3. Once their tasks show `completed` in `TaskList`, re-send `shutdown_request`. The teammate should now approve.
4. If they reject again citing the same incomplete work but `TaskList` shows their tasks as `completed`, the rejection is stale. Re-send once more; if rejected a third time, treat as unresponsive (see Scenario 2).

**Anti-patterns:**
- Re-sending `shutdown_request` immediately after rejection without checking `TaskList` first — forces a race condition.
- Calling `TeamDelete` while a teammate has active tasks — leaves work incomplete and may corrupt team state.
- Relying on memory instead of `TaskList` to assess what is done.

---

## 2. Teammate Unresponsive at Shutdown

**Situation:** The lead sends `shutdown_request` and receives no `shutdown_response` after approximately 2 minutes.

**Root cause:** The teammate's process may have stalled, or they are mid-task and not yet checking messages.

**Resolution:**
1. Call `TaskList` and inspect all tasks owned by the unresponsive teammate.
2. **If all their tasks show `completed`:** Their work is done. Non-response does not block `TeamDelete`. Proceed — call `TeamDelete`. Note the unresponsive teammate in your user summary.
3. **If tasks are `in_progress` or `pending`:** Do not delete the team yet. Re-send `shutdown_request` once. Wait another 2 minutes.
4. **If still unresponsive with incomplete tasks:** Claim the incomplete tasks yourself with `TaskUpdate` (set `owner` to your name), complete them, then call `TeamDelete`.

**Anti-patterns:**
- Calling `TeamDelete` while tasks are genuinely incomplete — leaves deliverables missing.
- Waiting indefinitely without checking `TaskList` to determine whether the work is actually done.
- Spawning a replacement teammate to finish the work when you (the lead) can handle it directly.

---

## 3. Task Stalled — No Owner

**Situation:** A task has been pending for a long time with no owner and empty `blockedBy`, meaning it is available but unclaimed.

**Root cause:** The intended owner didn't call `TaskList` after completing their previous task, or they received the task assignment but missed the notification.

**Resolution:**
1. Identify the intended owner (from Phase 3 planning decisions or task description).
2. Send a direct message to that teammate: "Task #N is unclaimed in TaskList — please call TaskList and claim it."
3. If unresponsive after one exchange (approximately 1–2 minutes), claim the task yourself with `TaskUpdate` and either complete it or reassign it to another qualified teammate.
4. If reassigning, message the new owner directly with the task ID and context.

**Anti-patterns:**
- Broadcasting to the entire team about one stalled task — wastes everyone's attention.
- Sending multiple messages to the intended owner before giving them time to respond.
- Leaving the task unclaimed and hoping someone picks it up — stalls the entire dependency chain.

---

## 4. Circular Task Dependency

**Situation:** `TaskCreate` succeeds but the team is deadlocked — Task A is blocked by Task B, and Task B is blocked by Task A (directly or transitively).

**Root cause:** The dependency chain was designed with mutual blocking, usually because two teammates each need the other's output as a prerequisite.

**Resolution:**
1. Identify the cycle by reviewing `blockedBy` relationships in `TaskList`.
2. Find the smallest interface contract between the two tasks — usually the shape of an API or data structure. This can be agreed on without either task being fully complete.
3. Split the blocking task: create a new "contract task" (e.g., "Define auth API shape — schema only, no implementation") with no blockers. Both dependent tasks block on this contract task instead of each other.
4. Update `addBlockedBy` on the dependent tasks to point to the new contract task and remove the circular reference.
5. Assign the contract task to the teammate best positioned to define the interface.

**Anti-patterns:**
- Removing blockers arbitrarily to unblock the cycle — causes integration failures when the actual dependency was real.
- Asking teammates to "just start and figure out the interface later" — replicates the coupling problem in code rather than resolving it in planning.
- Adding a third task that blocks both, creating a longer cycle instead of breaking it.

---

## 5. Messages Not Received (Background Agent Bug)

**Situation:** A teammate was spawned but never receives messages from the lead or other teammates. Messages appear to send successfully but produce no response.

**Root cause:** The teammate was spawned with `run_in_background: true`. Background agents cannot register with the Agent Teams system and are therefore invisible to the message bus — they cannot send or receive team messages.

**Resolution:**
1. Do NOT re-spawn the teammate with the same parameters — the replacement will have the same problem.
2. Check the `Agent` tool call that spawned this teammate. If `run_in_background: true` was set, that is the cause.
3. To recover their work: read the output file from their background task (the `output_file` path returned by the original `Agent` call). Relay relevant findings to the team manually.
4. Re-spawn the teammate without `run_in_background` (omit the parameter entirely). Provide them with the context from the background agent's output so they can continue without re-doing completed work.

**Anti-patterns:**
- Re-sending messages repeatedly to an unregistered background agent — messages are silently dropped, not queued.
- Treating the background agent's output file as equivalent to team communication — other teammates cannot read it; the lead must relay it.
- Using `run_in_background: true` for any teammate. This is never appropriate for Agent Teams — it is for standalone subagent tasks only.
