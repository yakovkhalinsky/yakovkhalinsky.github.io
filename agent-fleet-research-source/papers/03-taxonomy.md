# A Role-Oriented Taxonomy for Multi-Agent Systems

**Abstract.** We propose a lightweight, role-oriented taxonomy for long-running multi-agent operations fleets. Roles are defined by responsibility rather than by model architecture. We introduce six roles — Dispatcher, Builder, Runtime, Verifier, Researcher, and Archivist — describe their primary obligations, failure modes, and memory requirements, and show how the taxonomy exposes coordination anti-patterns. The goal is to give practitioners a way to reason about agent fleets without collapsing every agent into a generic assistant.

## 1. Why roles matter

A fleet of identical agents is easy to build and hard to operate. Every agent can do everything, so no agent owns anything. When a task fails, it is unclear whether the failure was in execution, routing, verification, or memory. A role-oriented fleet is harder to design but easier to debug because each role has a contract it must keep.

We do not propose that every fleet needs exactly six roles. We propose that the six roles below are a useful default for operations fleets, and that deviations from them should be explicit.

## 2. The six roles

### 2.1 Dispatcher

The Dispatcher decides who does what. Its contract is routing: given a goal and the current state of the fleet, it assigns the task to the right role, sets priority, and tracks ownership. Failures are silent misrouting, parallel assignment without merge logic, and failure to escalate when confidence is low.

### 2.2 Builder

The Builder produces durable artifacts: code, configuration, documentation, plans. Its contract is constructive correctness: the artifact should do what was asked, fit the surrounding context, and be reviewable by the Verifier. Failures are locally correct but globally wrong artifacts, incomplete changes, and drift between code and docs.

### 2.3 Runtime

The Runtime operates live systems. Its contract is safe execution: run the change, observe the result, and report back. Failures are destructive changes without rollback, loss of runtime state on restart, and divergence between intended and actual system state.

### 2.4 Verifier

The Verifier checks work before it is integrated or deployed. Its contract is independent validation: can this change be shown to satisfy its goal, fit constraints, and not break existing behaviour? Failures are checks that pass locally but fail end-to-end, checks that miss cross-role interactions, and rubber-stamp approvals.

### 2.5 Researcher

The Researcher gathers context, options, and trade-offs. Its contract is informed choice: before a decision is made, the relevant facts have been collected and presented. Failures are decisions made on stale context, missing alternatives, or overconfidence in a single source.

### 2.6 Archivist

The Archivist maintains durable memory: skills, runbooks, incident notes, patterns. Its contract is accessible history: any agent should be able to find what the fleet already knows. Failures are stale documentation, unsearchable notes, and knowledge silos that depend on one agent's private memory.

## 3. Cross-cutting concerns

Every role touches memory, ownership, observability, and fallback.

### 3.1 Memory responsibility

Each role has a different memory need. The Runtime needs fast recall of recent state and rollback plans. The Builder needs durable context for long-running changes. The Archivist needs the entire historical corpus. The Dispatcher needs current task ownership. A memory system designed for only one of these will starve the others.

### 3.2 Ownership

Ownership is the answer to the question: if this fails, who is responsible? The Dispatcher owns routing. The Builder owns the artifact. The Runtime owns the live state. The Verifier owns the gate. Ownership must survive restarts and hand-offs, which is why it must be recorded in shared memory rather than held in-process.

### 3.3 Observability

A role without observable output cannot be debugged. Each role should emit decisions, not just results: the Dispatcher logs why it chose a role; the Verifier logs why it passed or failed a change; the Archivist records what was updated and why. These decision logs are the raw material for incident review.

### 3.4 Fallback behaviour

What happens when a role is unavailable or wrong? The Dispatcher escalates to a human. The Verifier blocks forward motion. The Runtime runs a rollback plan. The Archivist falls back to a canonical, immutable artefact. Fallback behaviour should be explicit, not emergent.

## 4. Role-to-pattern mapping

| Role | Key pattern | Common anti-pattern |
|---|---|---|
| Dispatcher | Explicit routing with confidence threshold | Silent misrouting based on keyword matching |
| Builder | Reviewable artefacts with ownership | Locally correct, globally wrong changes |
| Runtime | Durable state and rollback plan | Destructive changes without recovery path |
| Verifier | Independent end-to-end validation | Checks that pass locally and fail in production |
| Researcher | Gathered context before decision | Decisions made on stale or partial context |
| Archivist | Searchable, versioned fleet memory | Documentation drift and knowledge silos |

## 5. How the taxonomy exposes anti-patterns

The taxonomy makes several anti-patterns visible:

- **Role collapse.** Two roles are merged into one agent. The result is a conflict of interest: the Builder cannot be the only Verifier of its own work.
- **Missing Dispatcher.** Tasks are assigned by implicit convention. The result is uneven load, missed hand-offs, and no escalation path.
- **Verifiability gap.** The Verifier is present but cannot inspect the Builder's output or the Runtime's live state. The role exists on paper but not in practice.
- **Memory blindness.** The Archivist is not connected to the other roles, so the fleet repeats mistakes because no one can find previous resolutions.

## 6. Comparison with existing frameworks

AutoGen and CrewAI use roles as personas or job descriptions. DSPy uses modules and optimisers. Our contribution is not to invent roles but to use them as operational contracts. A role is not a prompt wrapper; it is a set of obligations that other roles can rely on.

## 7. Conclusion

A role-oriented taxonomy is a design tool. It forces the designer to decide who is responsible for routing, construction, execution, validation, research, and memory. Those decisions are where most fleet failures originate. By naming the roles and their contracts, we make the failures visible before they become incidents.

## References

- [Agentic Strategies and Patterns Catalog](../artifacts/agentic-patterns.md)
- [Incident Archetypes](../artifacts/incident-archetypes.md)
- [Role Taxonomy](../artifacts/role-taxonomy.md)
- [Bibliography](../artifacts/bibliography.md)
