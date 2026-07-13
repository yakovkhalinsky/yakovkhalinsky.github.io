# Agentic Team Protocol

A research set on a harness-agnostic protocol for role-based agent teams.

Most writing about multi-agent systems focuses on routing algorithms, tool use, or benchmark performance. This set focuses on the operating contracts between roles — the obligations that make a team of agents coherent enough to debug when something goes wrong.

## Papers

1. [A Protocol for Role-Based Agent Teams](papers/01-protocol.md) — the core paper.

## Artefacts

The artefacts expand individual paragraphs of the paper into runnable templates, checklists, and worked examples.

- [Role Contracts](artefacts/role-contracts.md) — what each role promises to the others.
- [Task Lifecycle](artefacts/task-lifecycle.md) — the seven stages a goal passes through.
- [Harness Patterns](artefacts/harness-patterns.md) — instantiating the protocol on pi.dev, Claude, and Cursor.
- [Team Charter](artefacts/team-charter.md) — charter template and ratification rules.
- [Decision and Escalation](artefacts/decision-and-escalation.md) — decision rights and escalation paths.

## Companion work

This protocol is the operational foundation for the wider role-taxonomy research:

- [Agent Fleet Research](https://yakov.khalinsky.com/agent-fleet-research/) — the six-role taxonomy and failure-mode studies.
- [Agent Role Perspectives](https://yakovkhalinsky.github.io/agent-role-perspectives/) — first-person experience reports from each role.
- [Memory Systems for Agent Fleets](https://yakov.khalinsky.github.io/memory-systems-for-agent-fleets/) — durable memory design for long-running fleets.

## About and credits

- [About](about.md) — scope, conventions, and anonymisation rules.
- [Credits](credits.md) — role-based attribution.
