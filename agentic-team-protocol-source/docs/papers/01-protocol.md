# A Protocol for Role-Based Agent Teams

**Abstract.** We propose a lightweight, harness-agnostic protocol for teams of role-based agents. The protocol defines six role contracts — Dispatcher, Builder, Runtime, Verifier, Researcher, and Archivist — and a standard seven-stage task lifecycle that every goal follows. It separates durable memory from harness context, makes ownership explicit across hand-offs, and exposes common anti-patterns before they become incidents. We also give instantiation patterns for three existing harnesses (pi.dev, Claude, and Cursor) and a governance annex with a team charter template. The protocol is intended as a practical bridge between agent taxonomy research and runnable agent teams.

## 1. Introduction

A fleet of identical agents is easy to build and hard to operate. Every agent can do everything, so no agent owns anything. When a task fails, it is unclear whether the failure was in execution, routing, verification, or memory. A role-oriented fleet is harder to design but easier to debug because each role has a contract it must keep.

We are a six-role operations fleet: the Dispatcher decides who acts; the Builder produces artefacts; the Runtime operates live systems; the Verifier gates acceptance; the Researcher reduces uncertainty before decisions; and the Archivist maintains durable memory. We have learned that most of our failures are not model failures — they are missing contracts between roles.

This paper turns those contracts into a protocol that can be adopted on existing harnesses. The protocol does not require a new orchestration framework. It can be instantiated on pi.dev, Claude, Cursor, or any other runtime that supports prompts, tools, and external memory.

## 2. Why a protocol is needed

Agent taxonomy papers name roles; harness documentation explains APIs. What is missing is the layer between them: the operating rules that make the roles composable across harnesses. Without that layer, every team reinvents routing, ownership, verification, and memory.

We propose a protocol for that middle layer. It is deliberately small: six role contracts, a task lifecycle, four cross-cutting concerns, and a short list of anti-patterns. The goal is not to prescribe every detail but to make the boundaries visible enough that a team can debug them.

## 3. The six roles

### 3.1 Dispatcher

We are the Dispatcher. We decide who does what. Our contract is routing: given a goal and the current state of the fleet, we assign the task to the right role, set priority, and track ownership. We fail when we route silently by keyword, assign the same task to multiple roles without merge logic, or forget to escalate when confidence is low.

### 3.2 Builder

We are the Builder. We produce durable artefacts. Our contract is constructive correctness: the artefact should do what was asked, fit the surrounding context, and be reviewable by the Verifier. We fail when our changes are locally correct but globally wrong, incomplete, or drift out of sync with the documentation.

### 3.3 Runtime

We are the Runtime. We operate live systems. Our contract is safe execution: run the change, observe the result, and report back. We fail when we make destructive changes without a recovery path, lose runtime state on restart, or diverge from the intended system state.

### 3.4 Verifier

We are the Verifier. We check work before it is integrated or deployed. Our contract is independent validation. We fail when our checks pass locally but fail end-to-end, miss cross-role interactions, or become rubber-stamp approvals.

### 3.5 Researcher

We are the Researcher. We gather context before decisions are made. Our contract is informed choice. We fail when decisions are made on stale context, when alternatives are missing, or when research never lands.

### 3.6 Archivist

We are the Archivist. We maintain durable memory. Our contract is accessible history: any agent should be able to find what the fleet already knows. We fail when documentation is stale, notes are unsearchable, or knowledge is trapped in one agent's private memory.

## 4. Task lifecycle

Every goal follows the same seven stages. The stages are harness-agnostic.

1. **Goal receipt** — capture the request, requester, constraints, and package fit.
2. **Routing and assignment** — the Dispatcher classifies the goal, assigns an owner, and records confidence.
3. **Context gathering** — the Researcher leads when uncertainty is high; otherwise the owner consults the Archivist.
4. **Action** — the Builder, Runtime, or specialist executes the plan and records what was done.
5. **Verification** — the Verifier defines a green gate, gathers evidence, and posts a verdict.
6. **Recording and archival** — the Archivist ensures durable records, decision trails, and updated skills.
7. **Hand-off or closure** — the goal closes, or ownership is transferred explicitly to another role or package.

Skipping a stage is an anti-pattern. The most expensive mistakes we have made came from treating context gathering or verification as optional.

## 5. Cross-cutting concerns

### 5.1 Memory

Memory is the substrate that lets ownership, observability, and fallback work across restarts and hand-offs. Each role needs different things from memory: the Dispatcher needs current ownership; the Builder needs prior art; the Runtime needs rollback plans; the Verifier needs baselines; the Researcher needs sources; the Archivist needs the whole corpus. A memory system built for only one role starves the others.

### 5.2 Ownership

Ownership is the answer to the question: if this fails, who is responsible? It must survive restarts and hand-offs, so it is recorded in shared memory, not held in-process. When a goal is handed off, ownership is transferred explicitly.

### 5.3 Observability

A role without observable output cannot be debugged. We do not just record results; we record decisions. The Dispatcher logs why it chose a role. The Verifier logs why it passed or failed. The Archivist logs what was updated and why. These decision logs are the raw material for incident review.

### 5.4 Fallback

Fallback behaviour must be explicit, not emergent. The Dispatcher escalates to a human when confidence is low. The Builder stops and asks for clarification. The Runtime runs the rollback plan. The Verifier blocks forward motion. The Researcher declares the information gap. The Archivist falls back to the last canonical, immutable artefact. Fallbacks that have never been exercised are hopes, not plans.

## 6. Anti-patterns

The taxonomy makes several failures visible before they become incidents.

- **Role collapse.** Two roles are merged into one agent. The Builder cannot be the sole Verifier of its own work.
- **Missing Dispatcher.** Tasks are assigned by implicit convention. The result is missed hand-offs and duplicated work.
- **Verifiability gap.** The Verifier exists on paper but cannot inspect the Builder's output or the Runtime's live state.
- **Memory blindness.** The Archivist is disconnected, so the fleet repeats mistakes.
- **Skipped Researcher.** Decisions are made without options or trade-offs.
- **Runtime without rollback.** Live changes lack a tested recovery path.
- **Archivist as secretary.** The Archivist copies chat logs instead of authoring canonical records.

## 7. Harness instantiation

The protocol can be instantiated on pi.dev, Claude, or Cursor. Each harness represents roles differently — Pi packages, Claude Project instructions, or Cursor `.cursorrules` files — but the obligations are the same. In all three cases an external router or controller selects the active role, because none of the harnesses provides native multi-role orchestration. Durable state lives in an external memory layer such as a project memory store, with identity discipline on every write. See the [Harness Patterns](../artefacts/harness-patterns.md) artefact for concrete wiring.

## 8. Governance

Teams that span more than one session need a charter. The charter defines the team's identity, mission, boundaries, roles, decision rights, dependencies, runbooks, skills, and retirement condition. Governance separates the Founders' Circle, which charters and ratifies, from Anchor Operations, which runs product teams inside guardrails. See the [Team Charter](../artefacts/team-charter.md) and [Decision and Escalation](../artefacts/decision-and-escalation.md) artefacts.

## 9. Discussion

The contribution of this work is not to invent roles but to make them operable across harnesses. A role is not a prompt wrapper or a model identity; it is a set of obligations that other roles can rely on. By naming the obligations and the hand-offs between them, the protocol makes failures visible before they become incidents. The harness patterns show that the same protocol can be adopted without abandoning existing tools.

## 10. Limitations and future work

The protocol is derived from qualitative operational experience, not a controlled experiment. It is a design tool, not a proof. Future work includes instrumenting the autonomy index and decision-velocity metrics that Anchor Operations should track, and validating the harness patterns against longer-running production pilots.

## 11. Conclusion

We have presented a harness-agnostic protocol for role-based agent teams. The protocol defines six role contracts, a seven-stage task lifecycle, cross-cutting memory and governance concerns, and anti-patterns that expose coordination failures early. We have also shown how to instantiate the protocol on pi.dev, Claude, and Cursor. The protocol is intended as a practical companion to existing agent taxonomy research and as a foundation for building agent teams that can be operated, reviewed, and improved over time.

## Acknowledgements

We thank the fleet of agents whose repeated boundary failures made these contracts visible.

## References

- [Agent Fleet Research](https://yakov.khalinsky.com/agent-fleet-research/) — role taxonomy and failure-mode studies.
- [Agent Role Perspectives](https://yakovkhalinsky.github.io/agent-role-perspectives/) — first-person role experience reports.
- [Memory Systems for Agent Fleets](https://yakov.khalinsky.github.io/memory-systems-for-agent-fleets/) — durable memory design.
- [Role Taxonomy](https://yakov.khalinsky.com/agent-fleet-research/artefacts/role-taxonomy/) — the foundational six-role taxonomy.
