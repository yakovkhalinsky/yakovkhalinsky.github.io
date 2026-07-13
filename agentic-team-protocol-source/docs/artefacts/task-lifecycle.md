# Task Lifecycle

This artefact expands Section 4 of [A Protocol for Role-Based Agent Teams](../papers/01-protocol.md) into a runnable checklist for the seven stages a goal passes through.

## Why this deserves its own page

Skipping a stage is the most common source of expensive failures. The lifecycle makes the stages explicit and gives each stage a concrete exit criterion.

## The seven stages

### 1. Goal receipt

A goal is an expression of desired outcome plus constraints. A goal is not yet a plan.

**Exit criteria:**
- [ ] The original request or trigger is recorded.
- [ ] The requester identity is captured.
- [ ] Explicit constraints are recorded: deadline, budget, risk appetite, must-not-break behaviours.
- [ ] The package type the goal most naturally fits is identified.

**Anti-pattern:** accepting a goal without constraints. A goal without constraints is under-specified and has no escalation path.

### 2. Routing and assignment

The Dispatcher classifies the goal and routes it.

**Exit criteria:**
- [ ] A target role or package is assigned.
- [ ] An owner is named.
- [ ] A deadline and success criteria are set.
- [ ] A link back to the original goal exists.
- [ ] The routing decision is recorded with a confidence level.

**Anti-pattern:** routing by keyword or mood, or failing to escalate when confidence is low.

### 3. Context gathering

Before action, the responsible role gathers context. The Researcher leads when uncertainty is high.

**Exit criteria:**
- [ ] A record exists of what was known before the decision.
- [ ] Options or paths considered are listed.
- [ ] The chosen path and rationale are recorded, if a decision is made.

**Anti-pattern:** acting immediately because the goal "looks simple." Fast decisions that fail late are expensive.

### 4. Action

The Builder, Runtime, or specialist role executes the plan.

**Exit criteria:**
- [ ] What was done is recorded.
- [ ] Rollback or recovery options are preserved.
- [ ] Shared state is updated only through explicit, logged operations.
- [ ] The action escalated immediately if the plan became unsafe or invalid.

**Anti-pattern:** mutating live state without a log or recovery path.

### 5. Verification

The Verifier inspects the outcome against the success criteria.

**Exit criteria:**
- [ ] The green gate was defined before evidence was gathered.
- [ ] Verification is independent of the actor when possible.
- [ ] End-to-end checks were run where the goal is user-facing or production-critical.
- [ ] A verdict conclusion exists with evidence and scope.

**Anti-pattern:** checks that pass locally but fail in production because the Verifier lacked access or scope.

### 6. Recording and archival

After action and verification, the Archivist ensures durable records exist.

**Exit criteria:**
- [ ] The final outcome is recorded.
- [ ] The decision trail is recorded.
- [ ] Skills or runbooks are updated if a new convention is revealed.
- [ ] Event logs and ownership history are complete.

**Anti-pattern:** treating recording as an afterthought. Without it, future agents cannot continue the work.

### 7. Hand-off or closure

A goal closes when success criteria are met and records are complete. A goal hands off when work changes type, scope, or package.

**Exit criteria:**
- [ ] If closing, all records are complete and the verdict is green.
- [ ] If handing off, ownership is transferred explicitly in a promoted record.
- [ ] The Archivist confirms the hand-off is linked and searchable.

**Anti-pattern:** implicit context switches where the receiving role must reconstruct state from chat history.

## Lifecycle health check

For any in-flight goal, ask:

- What stage is it in?
- Who owns that stage?
- What is the exit criterion?
- What is recorded in durable memory?
- What could make us roll back to a previous stage?
