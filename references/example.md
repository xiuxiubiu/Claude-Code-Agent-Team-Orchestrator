# Worked Example: REST API Team

**Mission:** Build a REST API for a task management app with JWT auth, input validation, and API documentation.

---

## Phase 1: Role Justification Table

| Role | Unique Value | Cost of Omission | Deliverables |
|------|-------------|-------------------|--------------|
| `backend-dev` | Implements endpoints, data models, and business logic — the core artifact. | No API gets built. | 1. Express routes for `/tasks` CRUD. 2. Prisma schema + migrations. 3. JWT auth middleware. 4. Zod input validation. |
| `security-reviewer` | Identifies auth flaws, injection vectors, and insecure defaults a focused developer misses. | Auth bypass or token leakage ships to production. | 1. Threat model for auth flow. 2. Middleware chain review. 3. Pentest checklist with pass/fail per endpoint. |
| `doc-writer` | Produces accurate API documentation from implemented code, not from assumptions. | Users cannot integrate; support burden increases. | 1. OpenAPI 3.1 spec. 2. README with quickstart and auth examples. 3. Error code reference table. |

**Challenger role:** `security-reviewer` inherently provides adversarial review of the auth domain — explicit challenger waived for this 3-person team. All other roles satisfy "Minimum Viable Team" (non-overlapping deliverables).

---

## Phase 2: Approval Gate (Standard Mode)

The lead calls `AskUserQuestion`:

> **Proposed team for task-management REST API:**
>
> | # | Teammate | subagent_type | Key Deliverable |
> |---|----------|--------------|-----------------|
> | 1 | backend-dev | general-purpose | CRUD endpoints + JWT auth |
> | 2 | security-reviewer | qa-expert | Threat model + pentest checklist |
> | 3 | doc-writer | technical-writer | OpenAPI spec + README |
>
> Team size: 3 (minimum viable). Phase 3 needed: **Yes** — auth contract between backend-dev and security-reviewer must be agreed before coding. Approve or suggest changes?

User responds: "Approved."

---

## Phase 3: Planning Round (One Exchange)

**backend-dev → security-reviewer:**
> I plan to use `jsonwebtoken` with RS256 and store the refresh token in an httpOnly cookie. Access token: 15min. Refresh: 7 days. The `/auth/login` endpoint returns both. Does this approach have gaps I should address before I start coding?

**security-reviewer → backend-dev:**
> Two concerns. First, bind the refresh token server-side with a `refresh_tokens` table (jti, userId, expiresAt, revoked columns) — without it, stolen tokens are valid 7 days with no kill switch. Second, set `SameSite=Strict` and add a `/auth/revoke` endpoint. I'll add both to the pentest checklist.

**Lead synthesis decision:**
One round was sufficient. Decision: backend-dev adds `refresh_tokens` table and `/auth/revoke` endpoint. Security-reviewer must test revocation. Doc-writer documents the revocation flow. Tasks created accordingly.

---

## Phase 4: TaskCreate Calls

**Task 1 — backend-dev:**
```
subject: "Implement CRUD endpoints and JWT auth with refresh token revocation"
description: "Build Express routes for /tasks (GET, POST, PUT, DELETE) and /auth
(login, register, revoke). Prisma schema with refresh_tokens table (jti, userId,
expiresAt, revoked). RS256 tokens; access: 15min; refresh: 7d httpOnly
SameSite=Strict cookie. Zod validation on all request bodies.
Acceptance: correct status codes; auth middleware rejects expired/revoked tokens."
```

**Task 2 — security-reviewer (blocked by Task 1):**
```
subject: "Security review of auth flow and all endpoints"
description: "Threat model: token theft, replay attacks, injection, privilege
escalation. Pentest checklist for every endpoint. Verify refresh revocation
works on logout and password change. Verify SameSite and httpOnly flags.
Output: pass/fail table per endpoint with remediation notes for failures."
addBlockedBy: [1]
```

**Task 3 — doc-writer (blocked by Task 1):**
```
subject: "Write OpenAPI spec and README documentation"
description: "Generate OpenAPI 3.1 spec from implemented routes (read actual code,
not the plan). Include auth flow with refresh/revoke examples. README: quickstart
(install, env, run), auth walkthrough (register → login → refresh → revoke),
error code reference. Acceptance: spec validates with swagger-cli."
addBlockedBy: [1]
```

---

## Exception-Driven Communication Anti-Patterns

**DO NOT: Routine check-ins**
> backend-dev → team-lead: "Just wanted to update you — I'm halfway through the CRUD endpoints, everything going smoothly."

Nothing has changed. No action needed. Task status belongs in `TaskUpdate`, not messages.

**DO NOT: FYI broadcasts**
> doc-writer → broadcast: "FYI, I chose Swagger UI over Redoc for the API docs."

This is entirely within doc-writer's domain. No teammate can act on it. Use broadcast only for blockers that affect everyone.

**DO NOT: Seeking approval for own decisions**
> backend-dev → team-lead: "Should I use `zod` or `joi` for input validation?"

This was delegated to backend-dev. Tactical library choices within your deliverable are yours to make.
