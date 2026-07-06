# Failure Modes in Long-Running Agent Collaboration

**Abstract.** We report on the failure modes observed in a long-running multi-agent operations fleet. The failures are organised into five classes: memory and identity, dispatch and routing, build/deploy and runtime, documentation and coordination, and evaluation and metrics. For each class we give anonymised examples, show how failures cascade across the six roles, and propose containment strategies. The paper is intended as a companion to field-study experience reports and as groundwork for an evaluation protocol.

## 1. Introduction

A single agent that fails is usually easy to diagnose: the wrong answer, the bad tool call, the missing context. A multi-agent fleet that fails is harder because the failure migrates. A Runtime agent restarts and loses identity; a Builder writes a change the Verifier cannot validate; the Dispatcher routes a task to an agent that has stale memory of a previous run. The problem is no longer in one model; it is in the contract between agents.

This paper collects the failure modes we observed, names them, and groups them by the contract they violate. We do not claim the list is exhaustive. We claim that these patterns recur and that they are preventable once they are visible.

## 2. Method

We used the same anonymised incident notes that fed the field study, but this time we coded each incident by the contract it broke rather than by the role involved. The result is five failure classes. Within each class we identify composite failure chains: sequences in which one small breach causes a second, more damaging breach elsewhere in the fleet.

## 3. Failure classes

### 3.1 Memory and identity failures

The most expensive failures involved shared memory. A default identity bucket caused the Archivist to write notes that the Runtime could not see. A session-scoped recall missed facts from a different session even though the user was the same. A restart reset the agent's identity, so it reintroduced itself to other agents as a new peer.

These failures share a common shape: the memory contract promised continuity, but the implementation silently partitioned or reset the data. The symptom was not a crash but a gradual divergence of understanding between agents.

### 3.2 Dispatch and routing failures

A Dispatcher routed a security task to a Builder because the task description contained the word "patch". The Builder produced code; the Verifier, who should have reviewed it, was never called. The task completed and was wrong.

Other dispatch failures were subtler: the Dispatcher had a confidence threshold that skipped the Researcher for ambiguous tasks, or it sent the same task to two agents in parallel without a merge strategy. In every case the failure was a routing error masquerading as an agent error.

### 3.3 Build, deploy, and runtime failures

The Builder produced a change that passed all build-time checks but broke in the target environment. The Runtime rolled it out and then could not roll back because the rollback plan was not stored durably. A configuration drift between two services caused a volume claim to bind to the wrong node.

These failures are familiar to operators of any distributed system. In a fleet they are worse because the Verifier may run checks against one environment while the Runtime deploys to another, and the Archivist may document a procedure that no longer matches the live system.

### 3.4 Documentation and coordination failures

The Archivist maintained runbooks, the Builder kept README files, and the Researcher kept research notes. None referenced the others. A new agent joining the fleet had to read three inconsistent descriptions of the same workflow.

We also observed ritual failures: a hand-off checklist existed but was not enforced, a post-incident review was written but never re-read, and a daily status message became noise that agents learned to ignore. Coordination rituals that are not maintained become worse than no rituals at all because they create false confidence.

### 3.5 Evaluation and metric failures

The fleet optimised for task-completion count. The result was small, safe tasks completed quickly and large, risky tasks avoided or split into pieces that each counted as completion. A metric designed to track progress became the goal.

We also saw identity-preservation failures go unmeasured because there was no test that asked whether an agent still recognised its own previous decisions. Without that measurement, a memory regression could persist for weeks.

## 4. Composite failure chains

The most damaging incidents were chains. For example:

1. A Runtime restart reset the agent identity.
2. The Dispatcher treated it as a new peer and assigned onboarding tasks.
3. The new instance accepted a task that the original instance had already partially completed.
4. The Builder overwrote the partial work, producing a regression.
5. The Verifier passed the change because its checks did not include the deleted context.

Breaking the chain at any point would have prevented the regression: durable identity, a task-ownership registry, a merge-aware Verifier, or end-to-end context checks.

## 5. Containment strategies

Across all classes, the same containment strategies appeared:

- **Explicit identity.** Every memory write, task assignment, and verification must name who is acting and on whose behalf.
- **Durable ownership.** Tasks and artifacts must have an owner that survives restarts, not just an in-memory assignment.
- **Cross-role smoke tests.** A change that affects multiple roles should be exercised end-to-end, not role-by-role.
- **Immutable audit traces.** Who decided what, when, and why must be recoverable after the fact.
- **Metric review.** Fleet-level metrics should be reviewed periodically to ensure they still measure the intended outcome.

## 6. Relation to the rest of the set

These failure classes motivated the later work on fleet memory. Memory and identity failures are addressed by the explicit identity and async semantics paper. Dispatch and routing failures are addressed by the role taxonomy. Documentation and coordination failures are addressed by the memory-centric collaboration position paper. Evaluation and metric failures are addressed by the evaluation protocol.

## 7. Threats to validity

This is an observational study from one fleet. The incidents are anonymised and generalised, but the coding was done by the same team that operated the fleet, so hindsight bias is likely. The taxonomy is provisional; other fleets may see failure classes we did not.

## 8. Conclusion

Multi-agent fleet failures are rarely model failures in isolation. They are contract failures between agents: identity, ownership, routing, documentation, and measurement. Making these contracts explicit, durable, and observable is the most effective way to reduce the frequency and severity of incidents.

## References

- [Agentic Strategies and Patterns Catalog](../artifacts/agentic-patterns.md)
- [Incident Archetypes](../artifacts/incident-archetypes.md)
- [Role Taxonomy](../artifacts/role-taxonomy.md)
- [Bibliography](../artifacts/bibliography.md)
