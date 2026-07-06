# Agentic Strategies and Patterns Catalog

A catalog of recurring strategies, good patterns, and anti-patterns observed in a long-running multi-agent operations fleet. Each entry is derived from anonymized incident archetypes and operational experience.

The catalog is organized by the concern it addresses: dispatch, memory, verification, build/deploy, and coordination rituals. It is intended to feed the paper set and the role taxonomy.

## 1. Dispatch patterns

### Pattern 1.1: Role-based routing

**Statement:** Assign work to agents by responsibility, not by availability or message proximity.

**How it works:** A Dispatcher reads the task domain and routes to the agent whose role owns that domain: Builder for code, Runtime for infrastructure, Verifier for correctness gates, Researcher for options, Archivist for memory and guidance.

**Why it helps:** Clear ownership prevents multiple agents from making partial edits to the same artifact. It also makes misrouting auditable.

**When it fails:** When the Dispatcher routes based on keyword matching without confidence (see anti-pattern 1.2), or when boundaries between roles are vague.

**Linked archetype:** confident-misrouting

### Pattern 1.2: Verifiable dispatch instruction

**Statement:** Every dispatch must include a concrete deliverable and a verification criterion.

**How it works:** Instead of asking an agent to “look at X,” the Dispatcher says: “Fix X so that test Y passes; do not commit until Z is green.”

**Why it helps:** Status updates can be faked or stalled; a verifiable deliverable forces closure.

**Linked archetype:** loop-without-progress

### Anti-pattern 1.1: Status-only check-ins

**Statement:** A coordination ritual that asks agents for status but not for a deliverable.

**Symptoms:** Agents report next steps repeatedly without completing them; the same blocker appears in every report.

**Consequence:** The fleet keeps moving without making progress.

**Linked archetype:** loop-without-progress

### Anti-pattern 1.2: Confident misrouting

**Statement:** A Dispatcher assigns work to a specialist based on assumptions that are not verified.

**Symptoms:** A build agent is asked to fix a runtime problem; a runtime agent is asked to edit code. The specialist proceeds anyway because the instruction does not allow declining.

**Consequence:** Subtle wrong outputs, wasted time, and blame diffusion.

**Linked archetype:** confident-misrouting

---

## 2. Memory and shared-state patterns

### Pattern 2.1: Externalize session state

**Statement:** Session affinity must survive replica restart and horizontal scaling.

**How it works:** Store session metadata in a shared service (e.g., Redis) with an in-memory fallback for single-process or degraded modes.

**Why it helps:** Agents can scale without losing conversation context; recovery becomes stateless.

**Linked archetype:** drifting-context

### Pattern 2.2: Explicit identity in shared tools

**Statement:** Tools that write shared memory must require the caller’s identity and the subject’s identity; defaults must not encode a single-agent assumption.

**How it works:** A memory tool rejects a write unless both `observer_id` and `observed_id` are supplied.

**Why it helps:** Prevents silent misrouting into the wrong memory bucket in multi-agent systems.

**Linked archetype:** silent-dependency-break

### Pattern 2.3: Local-first, shared-when-useful

**Statement:** Keep transient, per-agent state in local memory; promote only stable, cross-agent facts to shared memory.

**How it works:** Agents tag local memories as `shareable`, `draft`, `private`, or `local-only`. Only `shareable` facts are promoted to the shared store with source attribution.

**Why it helps:** Reduces noise in shared memory and protects sensitive or unverified context.

**Linked archetype:** silent-dependency-break

### Anti-pattern 2.1: Drifting context

**Statement:** Summarization or process-local storage loses constraints, identity, or goal information that later agents need.

**Symptoms:** A restarted agent defaults to a generic identity; a long thread drifts from the original goal after repeated summarization.

**Consequence:** Decisions are made with stale or incomplete context.

**Linked archetype:** drifting-context

### Anti-pattern 2.2: Silent dependency break

**Statement:** An agent changes shared state without notifying other agents that depend on it.

**Symptoms:** One agent writes to a different identity namespace, regenerates a shared secret, or changes a config file; other agents fail hours later with no obvious cause.

**Consequence:** Cascading failures, long debugging loops.

**Linked archetype:** silent-dependency-break

---

## 3. Verification patterns

### Pattern 3.1: Gate before forward motion

**Statement:** No fleet-approved change moves to the next phase until a Verifier signs off on the current phase.

**How it works:** Each phase has written pass/fail criteria. The Verifier runs the criteria and posts a green/red verdict before the Dispatcher advances.

**Why it helps:** Prevents accumulating unverified debt across phases.

**Linked archetype:** tool-success-mission-failure

### Pattern 3.2: Calibrate evaluation metrics to the task distribution

**Statement:** A success metric must distinguish between successful outcomes in the target workload.

**How it works:** If a scoring formula collapses to zero for common successful cases, rescale or re-parameterize it.

**Why it helps:** A metric that cannot distinguish success is worse than no metric.

**Linked archetype:** tool-success-mission-failure

### Anti-pattern 3.1: Tool success, mission failure

**Statement:** Each step succeeds, but the composed workflow fails because there is no end-to-end gate.

**Symptoms:** Template renders, build passes, tests pass, but the deployed system is broken; benchmark tasks score as failed despite producing correct outputs.

**Consequence:** False confidence; regressions discovered late.

**Linked archetype:** tool-success-mission-failure

### Anti-pattern 3.2: Surface-level drift

**Statement:** Docs, scripts, and code examples diverge from the actual system shape.

**Symptoms:** New adopters follow a tutorial that uses flat config keys while the loader expects nested keys; tool names differ between README and runtime; quickstart emits URLs for a component that was removed.

**Consequence:** First-try failures, support load, erosion of trust.

**Linked archetype:** surface-level-drift

---

## 4. Build / deploy / runtime patterns

### Pattern 4.1: Coupled P0 bundles ship together

**Statement:** When multiple fixes are tightly coupled, land them as one unit with a single merge/deploy/rollback plan.

**How it works:** The Builder writes a merge plan; the Runtime agent writes a deploy plan; the Verifier writes a rollback plan. The fleet reviews all three before execution.

**Why it helps:** Prevents partial landing where one fix is on `master` and the runtime is still down.

**Linked archetype:** loop-without-progress

### Pattern 4.2: Idempotent secret bootstrap

**Statement:** Live secrets must be created once and preserved across upgrades; rotation must be a separate, deliberate operation.

**How it works:** A bootstrap script creates missing secrets and skips existing ones. Routine upgrades do not pass secret values. A CI check verifies that every expected secret is present in the rendered manifest.

**Why it helps:** Stops deploy tools from accidentally invalidating credentials.

**Linked archetype:** ungoverned-secret-mutation

### Pattern 4.3: Render the desired state, never suppress it

**Statement:** In declarative deployment tools, always emit the resource manifest; use live lookups only to preserve existing values, not to gate emission.

**How it works:** Helm templates always render Secrets; `lookup` reads current values so the upgrade is a no-op; `helm.sh/resource-policy: keep` prevents deletion.

**Why it helps:** Declarative tools prune resources they no longer see. Suppressing a resource to “protect” it causes the exact failure it was meant to prevent.

**Linked archetype:** tool-success-mission-failure

### Anti-pattern 4.1: Ungoverned secret mutation

**Statement:** A helper script or automation regenerates credentials that downstream consumers already hold.

**Symptoms:** A render-values helper is run against an environment file; it regenerates API keys and passwords and applies them. Services start, but agents cannot authenticate.

**Consequence:** Auth outage, credential drift, manual recovery.

**Linked archetype:** ungoverned-secret-mutation

### Anti-pattern 4.2: Stale hand-off

**Statement:** One agent completes recovery or setup but leaves state bound to an environment that no longer exists.

**Symptoms:** A cluster is rebuilt, but storage volumes still point to the old node identity; a kubeconfig on a workstation is stale after cluster recreation.

**Consequence:** Stateful workloads cannot start; operators cannot see the cluster.

**Linked archetype:** stale-hand-off

---

## 5. Coordination and governance patterns

### Pattern 5.1: Single-owner skills and docs

**Statement:** Every piece of agent guidance has one canonical owner and location.

**How it works:** Skills carry frontmatter declaring the owner, canonical path, and related skills. Duplicates are retired or merged, not copied silently.

**Why it helps:** Prevents drift between copies; agents read the same guidance.

**Linked archetype:** surface-level-drift

### Pattern 5.2: Drive forward without permission stalls

**Statement:** The Dispatcher advances multi-phase work to the next gate, convening the fleet at milestones, and only escalates to the human on true blockers.

**How it works:** At the end of each phase, the Dispatcher asks: what is blocked, who owns it, and can the fleet decide? If yes, continue. If no, escalate.

**Why it helps:** Reduces human coordination load; keeps momentum.

**Linked archetype:** loop-without-progress

### Pattern 5.3: Post-mission review as ritual

**Statement:** After every significant migration or incident, the fleet convenes a review and updates skills, runbooks, or conventions.

**Why it helps:** Captures lessons before they decay; turns incidents into durable capability.

**Linked archetypes:** all

### Anti-pattern 5.1: Decisions without landing rituals

**Statement:** The fleet approves changes but does not define who merges, deploys, verifies, and rolls back.

**Symptoms:** P0 fixes exist on branches weeks after approval while `master` remains broken; everyone assumes someone else will land the work.

**Consequence:** Approved work rots; runtime and codebases diverge.

**Linked archetype:** loop-without-progress

---

## 6. Pattern index by paper

| Pattern / Anti-pattern | Paper 1 (field study) | Paper 2 (anti-patterns) | Paper 3 (taxonomy) | Paper 4 (position) | Paper 5 (evaluation) |
|---|---|---|---|---|---|
| 1.1 Role-based routing | ✓ | ✓ | ✓ | | |
| 1.2 Verifiable dispatch instruction | ✓ | ✓ | ✓ | | |
| 1.1 Status-only check-ins | ✓ | ✓ | ✓ | | |
| 2.1 Externalize session state | ✓ | ✓ | | ✓ | |
| 2.2 Explicit identity in shared tools | ✓ | ✓ | | ✓ | |
| 2.3 Local-first, shared-when-useful | ✓ | ✓ | | ✓ | |
| 3.1 Gate before forward motion | ✓ | ✓ | | | ✓ |
| 3.2 Calibrate evaluation metrics | ✓ | | | | ✓ |
| 4.1 Coupled P0 bundles ship together | ✓ | ✓ | | | |
| 4.2 Idempotent secret bootstrap | ✓ | ✓ | | | |
| 4.3 Render the desired state | ✓ | ✓ | | | |
| 5.1 Single-owner skills and docs | ✓ | ✓ | ✓ | | |
| 5.2 Drive forward without permission stalls | ✓ | ✓ | ✓ | | |
| 5.3 Post-mission review as ritual | ✓ | ✓ | | | |

## 7. Pattern-to-archetype mapping

| Pattern / Anti-pattern | Incident archetype |
|---|---|
| 1.1 Role-based routing | confident-misrouting (counter) |
| 1.2 Verifiable dispatch instruction | loop-without-progress (counter) |
| 1.1 Status-only check-ins | loop-without-progress |
| 2.1 Externalize session state | drifting-context (counter) |
| 2.2 Explicit identity in shared tools | silent-dependency-break (counter) |
| 2.3 Local-first, shared-when-useful | silent-dependency-break (counter) |
| 3.1 Gate before forward motion | tool-success-mission-failure (counter) |
| 3.2 Calibrate evaluation metrics | tool-success-mission-failure (counter) |
| 4.1 Coupled P0 bundles ship together | loop-without-progress (counter) |
| 4.2 Idempotent secret bootstrap | ungoverned-secret-mutation (counter) |
| 4.3 Render the desired state | tool-success-mission-failure (counter) |
| 5.1 Single-owner skills and docs | surface-level-drift (counter) |
| 5.2 Drive forward without permission stalls | loop-without-progress (counter) |
| 5.3 Post-mission review as ritual | all |

*Use this catalog as the source of truth for examples and claims in the papers.*
