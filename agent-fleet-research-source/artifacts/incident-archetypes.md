# Incident Archetypes

Anonymized, generalizable incident patterns observed in long-running agent collaboration. Each archetype captures the interaction pattern, root cause, and lesson without exposing implementation specifics.

## 1. Stale hand-off

**Pattern:** One agent completes a task and passes it to another, but the context or state it passes is incomplete, stale, or bound to an environment that no longer exists.

**Root cause:** Shared memory writes and reads are not synchronized around task boundaries; recovery procedures ignore stateful resource lifecycle.

**Example archetype:** A runtime agent rebuilds a cluster, but storage volumes remain bound to the old node identity. Stateful agents then fail because their shared state references a location that no longer exists.

**Lesson:** Define explicit hand-off checkpoints and require the receiving agent to acknowledge completeness before proceeding; include stateful resource lifecycle in recovery playbooks.

## 2. Confident misrouting

**Pattern:** A dispatcher or pipeline assigns work based on assumptions that are no longer true, and downstream agents proceed with high confidence.

**Root cause:** Dispatch logic lacks a verification step or confidence threshold for environmental preconditions.

**Example archetype:** A build/deploy pipeline assumes an image registry is running and trusted. It is neither, so pulls fail. The failure is attributed to the wrong component until the precondition is explicitly checked.

**Lesson:** Verify environmental preconditions before routing work; make the precondition check visible and auditable.

## 3. Tool success, mission failure

**Pattern:** Each individual tool call or step succeeds, but the overall workflow fails because calls were composed in the wrong order or with inconsistent assumptions.

**Root cause:** Orchestration treats per-step status as workflow success; no end-to-end validation gate exists.

**Example archetype:** A configuration template is edited, a commit is clean, and a build passes, but the rendered manifest is syntactically broken because one value was removed and never replaced. The broken manifest is only discovered when a downstream deploy step fails.

**Lesson:** Verify end-to-end outcomes, not just per-step status codes or clean diffs.

## 4. Drifting context

**Pattern:** A long-running workflow gradually diverges from its original goal as intermediate results are summarized, reinterpreted, or stored in a way that loses constraints or identity.

**Root cause:** Compaction or session storage loses information that later agents need; replicas do not share session state.

**Example archetype:** A session resolver stores conversation state in-process. A request routed to a different replica or a restarted process silently drops the active session, defaulting to a generic identity and forgetting the task context.

**Lesson:** Preserve immutable goal or identity artifacts and externalize session state before advertising scale or recovery.

## 5. Silent dependency break

**Pattern:** An agent changes a shared resource another agent depends on, causing failures elsewhere hours or days later. The change itself appeared successful.

**Root cause:** No ownership model for shared state; no change notification; defaults hide the impact.

**Example archetype:** A shared memory tool defaults to a generic identity bucket. Most agents write there, but one agent writes with a different identity. Other agents querying with the generic identity never see those memories. Later, a deploy script regenerates a shared secret, invalidating seeded credentials without warning.

**Lesson:** Assign write ownership and identity; broadcast changes through a shared event log or schema; reject ambiguous calls rather than silently defaulting.

## 6. Loop without progress

**Pattern:** Two or more agents repeatedly exchange messages without converging on a decision or output. Status is reported, but no verifiable work is completed.

**Root cause:** The coordination ritual asks for updates instead of concrete, verifiable next actions; no deadline or escalation trigger.

**Example archetype:** A dispatcher asks a specialist for a status update each hour. The specialist replies with the next planned step but never performs it. After several cycles, the dispatcher realizes the original task is still stalled.

**Lesson:** Coordination check-ins must require a verifiable deliverable and a deadline; cap retries and escalate when no progress evidence appears.

## 7. Surface-level drift

**Pattern:** Documentation, scripts, and code examples diverge from the actual system shape. First-time adopters follow a path that looks correct but fails.

**Root cause:** Docs and scripts are not treated as testable artifacts; no single canonical source of truth.

**Example archetype:** Config examples use a flat key shape while the loader expects a nested shape; quickstart scripts emit URLs for a reverse proxy that was removed; tool names differ between docs and runtime. Each individual file is “correct” in isolation; together they are impossible to follow.

**Lesson:** Treat docs, scripts, and config examples as part of the product interface; keep one canonical example and verify it end-to-end.

## 8. Ungoverned secret mutation

**Pattern:** A helper script or automation regenerates credentials that downstream agents and services have already been seeded with. The mutation succeeds; auth breaks everywhere.

**Root cause:** Secret rendering and secret rotation are not separated; no guardrails prevent a render helper from overwriting live secrets.

**Example archetype:** A values-rendering script is run against an environment file. It regenerates API keys, database passwords, and signing secrets, then applies them. The API accepts the new keys, but agents and the database still hold the old ones, causing 401/403 across the fleet.

**Lesson:** Separate secret rendering from secret rotation; require explicit opt-in and a re-seeding plan before mutating live credentials.

## Relationship to the taxonomy

These archetypes map to roles and concerns in `role-taxonomy.md` and provide the empirical basis for the papers. Notably:

- **Stale hand-off** and **drifting context** are memory/session failures addressed by the Archivist role.
- **Confident misrouting** and **loop without progress** are dispatcher failures.
- **Tool success, mission failure** and **surface-level drift** are builder/verifier gaps.
- **Silent dependency break** and **ungoverned secret mutation** are runtime/archivist cross-cutting concerns.
