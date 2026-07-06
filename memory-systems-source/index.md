# Memory Systems for Agent Fleets

A research mini-site on memory systems for long-running multi-agent fleets: local-first runtime design, sync contracts, explicit identity and async semantics, and federated failure modes.

## Papers

1. [Designing a Local-First Memory Runtime for Agent Fleets](papers/07-local-first-runtime.md) — field study on moving from a heavy backend stack to a single edge process.
2. [A Taxonomy of Sync Strategies for Fleet Memory](papers/06-sync-taxonomy.md) — decision framework for fleet-memory sync contracts.
3. [Explicit Identity and Async Semantics in Agent Memory Tools](papers/08-explicit-identity.md) — tool-design argument for memory surfaces.
4. [Failure Modes in Federated Agent Memory](papers/09-federated-failure-modes.md) — incident experience report on federated-memory contract failures.

## Artifacts

- [Fleet Memory Patterns Catalogue](artifacts/fleet-memory-patterns.md)
- [Fleet Memory Incident Archetypes](artifacts/fleet-memory-archetypes.md)
- [Fleet Memory Bibliography](artifacts/fleet-memory-bibliography.md)

## Research process

- [Fleet Memory Publication Plan](plans/fleet-memory-publication-plan.md)
- [Fleet Memory Papers Review](reviews/fleet-memory-papers-review.md)

## Related research

The operational foundation for this research is documented in the [Agent Fleet Research](https://yakov.khalinsky.com/agent-fleet-research/) site, which studies multi-agent operations fleets and role-based collaboration.

## Building this site

```bash
cd /home/yakov/memory-research-site
.venv/bin/mkdocs serve
.venv/bin/mkdocs build --strict
```
