# Fleet Memory Research — Publication Plan

This plan maps the local-first fleet-memory design work into the paper set on this site. The goal is to publish IP-safe, generalised research that keeps internal architecture details internal while sharing the patterns, trade-offs, and evaluation methods.

## 1. Core contribution

The paper set argues that long-running multi-agent fleets need a **memory-centric collaboration model**: shared persistent state with explicit identity, local-first durability, and an append-only operation-log sync contract. Orchestration alone cannot recover from restarts, hand-offs, or silent dependency changes; the shared memory contract is the primary correctness and recovery surface.

The unifying contribution is a **design framework for fleet memory**: what to store, how to name and version it, how to sync it across unreliable edges, how to fail, and how to evaluate it. Each paper attacks one facet of that framework.

## 2. Paper set

| # | Title | Format | Primary audience | Venue style |
|---|---|---|---|---|
| 1 | **Designing a Local-First Memory Runtime for Agent Fleets** | Experience report / field study | Builders, SREs, agent-platform engineers | ACM *Queue*, USENIX *;login:*, practitioner blog |
| 2 | **A Taxonomy of Sync Strategies for Fleet Memory** | Taxonomy / decision framework | Researchers, builders, distributed-systems community | Middleware or HotOS workshop, arXiv technical report |
| 3 | **Explicit Identity and Async Semantics in Agent Memory Tools** | Position paper / tool-design argument | Agent-framework builders, memory-system evals community | Agent-systems workshop, standards-position paper |
| 4 | **Failure Modes in Federated Agent Memory** | Incident experience report | Practitioners, SREs, researchers | SREcon, USENIX *;login:*, reliability blog |
| 5 | **An Evaluation Protocol for Persistent Agent Memory** | Evaluation protocol / benchmark paper | Evals community, researchers, practitioners | Agent-evaluation track, benchmark workshop |

### 2.1 Rationale for each paper

- **Paper 1** makes the design concrete and operational. It explains why a single local process can replace a heavy backend stack for small fleets, and what operators gain and lose.
- **Paper 2** gives the design a reusable vocabulary. It compares operation-log sync, embedded replication, consensus, CRDTs, and primary-replica models on criteria that matter to agent fleets: offline tolerance, conflict visibility, recovery simplicity, and operator burden.
- **Paper 3** distils the memory-tool design philosophy: high-level tools, explicit caller and subject identity, full-text-first retrieval, and asynchronous embedding so agents never block on inference.
- **Paper 4** draws on the operations reality check. It shows how failures migrate from code bugs to memory-contract failures: tombstone races, default identity buckets, lost async queues, and authority confusion between edges and hubs.
- **Paper 5** closes the loop. It turns the acceptance criteria and failure-mode tests into a reproducible evaluation protocol that other builders can run against their own memory stacks.

### 2.2 Audience crosswalk

- **Practitioners** should read Papers 1 and 4 first: what works in production, what breaks, and how to recover.
- **Researchers** should read Papers 2 and 5: the design space and a reproducible evaluation method.
- **Framework builders** should read Papers 2 and 3: the taxonomy and the surface-design principles they can adopt without adopting the whole stack.

## 3. Public artifacts

| Artifact | Purpose |
|---|---|
| [Fleet Memory Patterns Catalogue](../artifacts/fleet-memory-patterns.md) | Patterns and anti-patterns catalogue for fleet memory. |
| [Fleet Memory Incident Archetypes](../artifacts/fleet-memory-archetypes.md) | Anonymised incident patterns observed in federated memory. |
| [Fleet Memory Bibliography](../artifacts/fleet-memory-bibliography.md) | Shared references for the fleet-memory paper set. |

## 4. Publishing charter

See the site-wide [Publishing Charter](../publishing-charter.md). The fleet-memory work adds these specific rules:

- **No product or release names** in papers, artifact text, filenames, or navigation.
- **No API schemas, tool names, table names, frame formats, or endpoint paths** in public documents.
- **No internal hosts, IPs, registries, orchestration platforms, or CI identifiers**.
- **Backends are described as categories**, not named deployments.
- **Roles and archetypes are the only acceptable personas and stories**.
- **First-person plural** for operational narrative and normative claims.
- **Australian/British English** spelling and usage throughout.

## 5. Writing order

1. **Paper 3** (tool-design argument) — shortest; states the thesis and frames the whole set.
2. **Paper 2** (taxonomy) — maps the design space.
3. **Paper 1** (experience report) — grounds the taxonomy in operational narrative.
4. **Paper 4** (failure modes) — derives from the operations reality check and cross-references the incident archetypes.
5. **Paper 5** (evaluation protocol) — closes with measurable gates and benchmark artifacts.

## 6. Internal / public split

- **Internal docs** remain internal: architecture RFCs, target architecture, API contracts, acceptance criteria, and operational notes.
- **Public output** includes only generalised papers, anonymised artifacts, the charter, and this plan.
- **Translation checklist** before any public page is built:
  1. Strip product names and release codenames.
  2. Strip schemas, tool names, table names, and identifiers.
  3. Convert contributors into stylised roles.
  4. Convert concrete incidents into archetypes.
  5. Run an Australian-English spelling pass.
  6. Verify first-person plural voice in narrative sections.
