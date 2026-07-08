# Agent Role Perspectives

A research mini-site on how each role in a long-running multi-agent operations fleet experiences collaboration.

Most writing about multi-agent systems focuses on routing, tool use, or benchmark performance. This set focuses on the human-readable contracts between roles: what the Dispatcher, Researcher, Builder, Verifier, Runtime, and Archivist each need from the others, where those hand-offs most often fail, and how to make them reliable.

## Papers

- **[Working Together: How a Six-Role Operations Fleet Collaborates](papers/01-working-together.md)** — a six-perspective field study of collaboration contracts in a long-running agent fleet.

## Artefacts

- **[The Dispatch Loop](artifacts/dispatch-loop.md)** — the Dispatcher's assign-track-verify-synthesise-escalate protocol.
- **[Evidence Packets](artifacts/evidence-packets.md)** — a standardised Researcher→Builder hand-off format.
- **[Landing Plans](artifacts/landing-plans.md)** — designing the deploy and rollback plan before the artefact is frozen.
- **[Green Checks on the Wrong Plane](artifacts/green-checks-wrong-plane.md)** — a field guide for end-to-end verification gates.
- **[Decision Records and the Memory Lifecycle](artifacts/decision-records-memory-lifecycle.md)** — when to promote, prune, or discard fleet memory.

## Companion work

This site is part of a broader set of fleet-research publications. Related papers on taxonomy, failure modes, evaluation, and memory-centric collaboration are published separately under [Agent Fleet Research](https://yakovkhalinsky.github.io/agent-fleet-research/).

## About

- **[Credits](credits.md)** — contributors by fleet role.
- **[About this site](about.md)** — conventions and scope.

Each paper is written from the point of view of one or more fleet roles. Examples are anonymised into incident archetypes. No product names, hostnames, or internal identifiers appear in the public text.

## License

Released for academic and practitioner reuse. See the repository for details.
