# Role Contracts

This artefact expands Section 3 of [A Protocol for Role-Based Agent Teams](../papers/01-protocol.md) into a reusable contract per role. Each contract states the obligation, required outputs, and failure modes.

## Why this deserves its own page

A role is not a prompt wrapper or a model identity. It is a set of obligations that other roles can rely on. When a team collapses two roles into one agent, or omits a role, these are the contracts that silently break.

## Dispatcher

**Obligation:** decides who does what.

**Required outputs:**
- A routable goal or task record.
- An explicit target role or package.
- Success criteria and deadline.
- A confidence level or escalation trigger.

**Failure modes:**
- Silent misrouting by keyword or mood.
- Parallel assignment without merge logic.
- Failure to escalate when confidence is below threshold.
- Routing to a role that is unavailable without a fallback plan.

## Builder

**Obligation:** produces durable, reviewable artefacts.

**Required outputs:**
- The artefact itself.
- A change summary that explains intent and scope.
- Links to requirements, constraints, and decisions.
- Merge or integration instructions where applicable.

**Failure modes:**
- Locally correct but globally wrong artefacts.
- Incomplete changes that leave the system in a partial state.
- Drift between code, configuration, and documentation.
- Building without reading prior art from the Archivist.

## Runtime

**Obligation:** operates live systems safely.

**Required outputs:**
- An execution plan with ordered steps.
- A rollback or recovery plan.
- Observed state before and after the change.
- Health or status evidence from the live system.

**Failure modes:**
- Destructive changes without a recovery path.
- Loss of runtime state on restart or hand-off.
- Divergence between intended and actual system state.
- Ungoverned mutation of secrets or stateful resources.

## Verifier

**Obligation:** validates work before it is accepted.

**Required outputs:**
- A clear verdict: green, red, or blocked.
- Evidence linked to the verdict.
- The scope of what was checked.
- A list of residual risks or follow-up checks.

**Failure modes:**
- Checks that pass locally but fail end-to-end.
- Checks that miss cross-role interactions.
- Rubber-stamp approvals under time pressure.
- Verifying only tool output instead of mission outcome.

## Researcher

**Obligation:** gathers context before decisions are made.

**Required outputs:**
- A summary of the question and why it matters.
- Sources, options, and alternatives considered.
- Trade-offs and confidence levels.
- Recommended next step or information gap.

**Failure modes:**
- Decisions made on stale or partial context.
- Missing alternatives or overconfidence in a single source.
- Research without a clear consumer or decision hook.
- Gathering information indefinitely without landing it.

## Archivist

**Obligation:** maintains durable, searchable fleet memory.

**Required outputs:**
- Canonical, versioned records.
- Searchable indices or namespaces.
- Links between related records.
- Updates to skills and runbooks after missions.

**Failure modes:**
- Stale documentation that misleads later agents.
- Unsearchable or fragmented notes.
- Knowledge silos that depend on one agent's private memory.
- Recording results without recording decisions and rationale.

## Checklist for new teams

- [ ] Each role has a dedicated prompt or instruction file.
- [ ] Each role's required outputs are defined before the first mission.
- [ ] Each role's failure modes are reviewed in a post-mission retrospective.
- [ ] No role is the sole judge of its own output.
