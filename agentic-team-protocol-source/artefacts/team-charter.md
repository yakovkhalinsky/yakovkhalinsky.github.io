# Team Charter

This artefact expands Section 8 of [A Protocol for Role-Based Agent Teams](../papers/01-protocol.md) into a charter template and ratification process.

## Why this deserves its own page

A team without a charter has no boundaries. The charter is the contract between the team, Anchor Operations, and the Founders' Circle. It is the first artefact the Archivist creates and the last reviewed at retirement.

## Charter template

```markdown
# Charter — <Team Name>

## Identity
- Team name:
- Package type: [Ad-hoc Response / Goal Squadron / Operations Watch / Scheduled Ritual / Overseer Cell]
- Parent team or Anchor Operations liaison:
- Charter version:
- Last reviewed:

## Mission
One sentence describing the outcome the team owns.

## Boundaries
- In scope:
- Out of scope:
- Shared interfaces owned by other teams:

## Roles and seats
| Seat | Role | Agent or persona | Primary obligation |
|------|------|------------------|--------------------|
|      |      |                  |                    |

## Decision rights
See the [Decision and Escalation](decision-and-escalation.md) artefact.

## Escalation paths
- Internal deadlock:
- Guardrail breach:
- Data-loss or outage risk:
- Founder sign-off required:

## Interfaces and dependencies
| Dependency | Direction | Contract | Owner in other team |
|------------|-----------|----------|---------------------|
|            |           |          |                     |

## Runbooks and skills owned
- Runbooks:
- Skills:
- Prompts:

## Status cadence
- Daily card to Anchor Operations: yes / no
- Weekly summary: yes / no
- Gate review schedule:

## Retirement condition
When and how this team's charter expires or is merged into another team.
```

## Ratification rules

A charter is not active until ratified. The durable ratification record must include:

- The version of the charter that was approved.
- The identity of each rater.
- The date of ratification.
- The mechanism used (merge approval, recorded vote, signed memo).
- Any explicit deferrals or open decisions.

The Archivist stores the ratification record and links it to the team charter. Unratified teams may do foundation work but may not begin production implementation until sign-off or explicit deferral is recorded.

## Governance layers

| Layer | Purpose | Example |
|---|---|---|
| Founders' Circle | Charter and ratify teams, guardrails, budget, major pivots. | Original fleet plus project owner. |
| Anchor Operations | Run product teams inside guardrails. | Dedicated meta team with six seats. |
| Product teams | Execute domain work and report upward. | Core Memory, Team Kit, Harness Integration. |

## Anchor Operations six seats

| Research role | Seat | Responsibility |
|---|---|---|
| Dispatcher / Lead | Chair | Cadence, conflict resolution, founder interface, status synthesis. |
| Builder | Engineering Lead | Shared infrastructure, CI/CD, cross-team API contracts. |
| Runtime / Operator | Operations Lead | Deployment, resourcing, budgets, vendors, legal. |
| Verifier | Quality Lead | Verification gates, health audits, autonomy metrics. |
| Researcher | Strategy Lead | Roadmap, OKRs, prioritisation, external context. |
| Governor / Archivist | Convention Lead | Charter hygiene, conventions, documentation standards. |

## Charter health check

- [ ] The mission is one sentence and describes an outcome, not an activity.
- [ ] Boundaries name what is out of scope as clearly as what is in scope.
- [ ] Every role has a named seat and a primary obligation.
- [ ] Escalation paths are specific, not "ask someone."
- [ ] Dependencies are mapped to contracts and owners.
- [ ] The retirement condition is explicit.
- [ ] The charter has a ratification record linked by the Archivist.
