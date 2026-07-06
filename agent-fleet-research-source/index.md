# Agent Fleet Research

A publishable research set on multi-agent operations, failure modes, role taxonomy, and evaluation.

## Papers

1. [Agentic Strategies in a Six-Agent Operations Fleet](papers/01-field-study.md) — field study of patterns and anti-patterns.
2. [Failure Modes in Long-Running Agent Collaboration](papers/02-failure-modes.md) — deeper incident experience report.
3. [A Role-Oriented Taxonomy for Multi-Agent Systems](papers/03-taxonomy.md) — framework paper.
4. [Memory-Centric Collaboration](papers/04-position-paper.md) — position paper.
5. [Toward an Evaluation Protocol for Persistent Multi-Agent Workflows](papers/05-evaluation-protocol.md) — evaluation protocol.

## Artifacts

- [Agentic Strategies and Patterns Catalog](artifacts/agentic-patterns.md)
- [Role Taxonomy](artifacts/role-taxonomy.md)
- [Incident Archetypes](artifacts/incident-archetypes.md)
- [Bibliography](artifacts/bibliography.md)

## How to read this set

- **Practitioners** should start with the field study and the patterns catalog.
- **Researchers** should start with the taxonomy and the position paper.
- **Evaluators** should start with the evaluation protocol.

All papers draw from the same empirical base: a long-running six-agent operations fleet with stable roles and shared persistent memory.

## Building the site

```bash
cd mini-site
mkdocs serve
mkdocs build
```
