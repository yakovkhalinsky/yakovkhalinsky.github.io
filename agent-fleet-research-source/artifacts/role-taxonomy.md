# Role Taxonomy for Multi-Agent Operations Fleets

A lightweight, role-oriented taxonomy derived from running long-lived agent fleets. Roles describe *responsibilities*, not necessarily one agent per role.

## Core roles

| Role | Responsibility | Key concerns |
|------|----------------|--------------|
| **Dispatcher** | Decides who does what, monitors progress, resolves ambiguity | Routing, prioritization, deadlock detection |
| **Builder** | Implements, modifies, and packages code or configurations | Reproducibility, dependency hygiene, rollback |
| **Verifier** | Checks correctness, runs tests, audits outputs | Coverage, false positives, integration gaps |
| **Researcher** | Gathers information, evaluates options, reduces uncertainty | Source quality, stale data, over-research |
| **Runtime** | Operates infrastructure, keeps services alive | Health checks, observability, incident response |
| **Archivist** | Maintains shared memory, context, continuity, and agent guidance. | Retrieval accuracy, privacy, forgetting curves, skill/doc drift |

## Cross-cutting concerns

- **Memory**: who reads, who writes, what is canonical
- **Ownership**: which role is accountable when a hand-off fails
- **Observability**: how the fleet knows its own state
- **Fallbacks**: what happens when a role is unavailable or wrong

## Open questions

- Should the taxonomy distinguish *human-in-the-loop* roles separately?
- How do roles map to existing frameworks (AutoGen, CrewAI, DSPy agents)?
- Can role failure be predicted or instrumented?
