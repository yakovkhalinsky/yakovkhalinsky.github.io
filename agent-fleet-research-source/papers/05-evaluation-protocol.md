# Toward an Evaluation Protocol for Persistent Multi-Agent Workflows

**Abstract.** We propose a lightweight evaluation protocol for multi-agent workflows that run over time and share persistent state. Existing benchmarks focus on single-turn correctness, tool use, or reasoning; they miss the problems that dominate long-running collaboration: continuity, identity, hand-offs, and recovery. We define evaluation criteria, propose tasks and metrics, and outline a pilot protocol. The goal is to give practitioners and researchers a way to measure whether a fleet can sustain collaborative work, not merely whether it can answer a question correctly.

## 1. What existing benchmarks miss

Most benchmarks treat an agent as a function: input prompt, output response. They measure reasoning, coding, or tool selection in isolation. That is useful for model development but insufficient for fleet evaluation.

A long-running workflow spans multiple turns, multiple agents, and multiple sessions. The hard questions are not "did this call return the right answer?" but "does the fleet still know what it decided yesterday?", "can agent B continue agent A's task?", and "does the fleet recover gracefully from a restart?".

## 2. Evaluation criteria

We propose five criteria for an evaluation protocol aimed at persistent multi-agent workflows.

### 2.1 Continuity

Can an agent recall the facts it needs from previous turns and previous sessions without re-prompting?

### 2.2 Identity preservation

After restart, hand-off, or re-assignment, does an agent recognise its own prior decisions and the decisions of its peers?

### 2.3 Correct hand-off

When a task moves from one role to another, is the relevant context transferred completely and accurately?

### 2.4 Recovery

After a simulated failure — an agent restart, a network partition, or a bad change — can the fleet return to a known-good state and continue?

### 2.5 End-to-end outcome

Does the fleet achieve the user-level goal, not merely the intermediate objectives?

## 3. Proposed tasks

We sketch a set of tasks designed to exercise the criteria. Each task is multi-step and multi-agent, and it requires shared state to persist across steps.

### 3.1 Resume after restart

A Builder is halfway through a change when its process restarts. A new Builder instance must pick up the task, read the plan from shared memory, and continue without re-deriving the design.

### 3.2 Cross-agent hand-off

A Researcher gathers requirements and stores them. A Builder reads them, produces a change, and stores the change. A Verifier reads the change and the requirements, then decides whether to approve. The Dispatcher tracks ownership throughout.

### 3.3 Rollback

A Runtime deploys a change that fails in production. The fleet must identify the last known-good state, roll back, and record the rollback decision in shared memory so the same mistake is not re-attempted.

### 3.4 Documentation update

After an incident, the Archivist must update the runbook and notify the Builder and Runtime that a procedure has changed. The next time a similar task is attempted, the fleet must follow the new procedure.

## 4. Metrics

For each task we suggest three kinds of metric:

- **Binary success:** did the fleet complete the task?
- **Contextual accuracy:** did the fleet recall the right facts at the right time, measured by a human or a reference trace?
- **Efficiency:** how many extra turns, re-prompts, or corrections were required compared to an oracle trace?

A metric is useful only if it does not game the criteria. For example, a continuity metric that rewards any recall, regardless of relevance, would encourage noisy memory. Metrics should be paired with adversarial probes: memories that look correct but are wrong in context.

## 5. Pilot protocol outline

The pilot protocol is intentionally simple:

1. Define the six roles and the memory surface.
2. Run each task three times: fresh, after a restart, and after a partial failure.
3. Record the binary outcome, the recall accuracy, and the number of corrective turns.
4. Score each run against the criteria, then aggregate.
5. Review failures as incident archetypes and feed them back into the protocol.

The protocol is not a benchmark leaderboard. It is a diagnostic tool for discovering where a fleet's memory and coordination contracts break.

## 6. Relation to the rest of the set

This protocol is grounded in the failure modes paper, the role taxonomy, and the position paper. The field study provides the observed behaviours that the tasks try to reproduce. The failure modes paper provides the classes of defect the protocol should catch. The role taxonomy defines the agents that participate in each task.

## 7. Conclusion

Evaluating multi-agent fleets requires moving beyond single-turn correctness. Continuity, identity, hand-off, recovery, and end-to-end outcome are the properties that matter for long-running collaboration. A lightweight protocol built around these criteria can reveal memory and coordination failures that traditional benchmarks miss.

## References

- [Agentic Strategies and Patterns Catalog](../artifacts/agentic-patterns.md)
- [Incident Archetypes](../artifacts/incident-archetypes.md)
- [Role Taxonomy](../artifacts/role-taxonomy.md)
- [Bibliography](../artifacts/bibliography.md)
