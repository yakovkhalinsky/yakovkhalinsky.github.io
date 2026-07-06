# Publishing Charter

The public research material in this directory is intended for external publication as a set of papers and a mini-site. It must contain no Eden-specific implementation details, intellectual property, or internal identifiers.

## Public naming conventions

- Do not use the name **Eden** in public papers or site copy.
- Use stylized role names for agents: **Dispatcher**, **Builder**, **Verifier**, **Researcher**, **Runtime**, **Archivist**.
- Refer to the system generically as a *multi-agent operations fleet* or *agent fleet*.

## Incident archetypes

Concrete incidents from internal operations should be transformed into *incident archetypes*:

- Remove dates, hostnames, and project names.
- Replace specific failures with their operational category.
- Keep only the interaction pattern, root-cause category, and lesson.

Example:

- Internal: *On 2024-11-03 the registry build failed because `pnpm run clean` removed the lockfile and the CI image cache was stale.*
- Public archetype: *A build agent invalidated its own dependency lockfile while the runner image cache was stale, causing a non-deterministic build failure.*

## Acceptable public content

- Roles and interaction patterns
- Failure-mode taxonomies
- Design principles and trade-offs
- Evaluation methods and metrics
- High-level architecture patterns (without Eden-specific wiring)

## Prohibited public content

- Product name, logo, or branding
- Source code, schemas, or API signatures
- Internal hostnames, IPs, cloud accounts, or deployment details
- Private conversations or identifiable user messages
- Commercial plans, pricing, or unreleased features

## Review process

Before any public document is published:

1. Author self-checks against this charter.
2. A second reviewer confirms no Eden-specific material remains.
3. Final approval by Yakov Khalinsky.
