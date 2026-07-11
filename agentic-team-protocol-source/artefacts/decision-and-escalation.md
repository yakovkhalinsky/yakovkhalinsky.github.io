# Decision and Escalation

This artefact expands Section 8 of [A Protocol for Role-Based Agent Teams](../papers/01-protocol.md) into a decision matrix, escalation triggers, and escalation package format.

## Why this deserves its own page

Decision rights and escalation paths are where most governance failures begin. A decision with two equal owners stalls. A risk without a route to a human festers. This page makes both explicit.

## Decision matrix

| Decision type | Primary owner | Must consult | Must inform | Escalation trigger |
|---|---|---|---|---|
| Which role or package owns a goal | Dispatcher | — | Anchor Operations if cross-team | Confidence below threshold; all specialists decline |
| Technical design inside a package | Builder | Runtime if deploy-related; Verifier if correctness-critical | Archivist if it changes conventions | Design crosses team boundary or guardrail |
| Operational change to live systems | Runtime | Builder if code change; Verifier before execution | Dispatcher; Archivist if runbook changes | No tested rollback plan; risk above threshold |
| Whether a gate is green | Verifier | — | Dispatcher; Builder if red | Verifier lacks access or scope |
| Which options to consider | Researcher | Dispatcher | Archivist | No alternatives available; deadline forces choice |
| Which conventions to adopt or change | Archivist | All affected roles | Anchor Operations Convention Lead | Cross-team convention change |
| Cross-team API or interface contract | Engineering Lead (Anchor) or team Builder | Runtime; Verifier | Archivist; affected product teams | Contract change breaches guardrail |
| Roadmap priority and OKRs | Strategy Lead (Anchor) | Chair; Engineering Lead | Founders' Circle if budget or pivot | Sibling deadlock or founder guardrail breach |
| Budget, vendor, legal, production deploy | Operations Lead (Anchor) | Chair; Founders' Circle | Product teams | Money, liability, or data-deletion risk |
| Charter, team creation, major pivot | Founders' Circle | Anchor Operations Chair | All affected teams | Founder deadlock |

## Decision ownership rules

1. **One decision, one owner.** Two equal owners is a stalled decision.
2. **The owner must have authority over the affected resources.** A Builder cannot decide deployment architecture without Runtime authority.
3. **Consultation is not veto power.** Those consulted provide input; the owner decides.
4. **Informed parties must be named.** "The team was told" is not enough.
5. **Escalation is a feature, not a failure.** Escalate when a decision crosses a charter or guardrail.

## Escalation triggers

Escalate immediately when any of the following occur:

1. Low-confidence routing.
2. Role deadlock.
3. Guardrail breach.
4. Data-loss or outage risk.
5. Repeated failure after two auto-resolution attempts.
6. Missing authority.
7. Founder sign-off required.

## Escalation path

| Level | Route | Response target |
|---|---|---|
| 1 — Internal | Owning role escalates to team Dispatcher or Overseer. | Within one status period. |
| 2 — Anchor Operations | Team Dispatcher escalates to Anchor Operations Chair. | Same day. |
| 3 — Founders' Circle | Anchor Operations Chair escalates with a sign-off package. | Within 48 hours for guardrail or risk issues. |
| 4 — Human intervention | Founders' Circle or project owner makes the final call. | As soon as human availability allows. |

## Escalation package

An escalation must include:

- The goal or decision being escalated.
- The options considered.
- The positions of each consulted role.
- The recommended default, if any.
- The specific question or authority being requested.
- The risk of waiting.

Do not escalate with "what should I do?" Escalate with "I recommend X unless you object, because Y, with risk Z."

## De-escalation

Once resolved, the Archivist records:

- The decision made.
- The rationale.
- Any guardrail or charter change required.
- Who communicated the decision back to the team.

The team then resumes normal operation. Unresolved escalations older than the response target are re-escalated to the next level.
