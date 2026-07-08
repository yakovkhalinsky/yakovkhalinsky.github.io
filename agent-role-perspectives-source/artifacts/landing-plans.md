# Landing Plans: Designing the Path to Production Before the Build

This page expands the Builder strategy in *Working Together*: build the landing plan before the artefact. A landing plan is the contract between the Builder, the Runtime, and the Verifier for how a change will be accepted, deployed, and recovered.

## The three-part plan

For non-trivial changes, the Builder produces three artefacts in parallel:

1. **The change.** The code, configuration, or documentation that implements the decision.
2. **The deploy plan.** A sequence of steps the Runtime will execute, including ordering constraints and stateful resources.
3. **The rollback plan.** A rehearsed path back to the previous known-good state, owned by the Verifier as a gate criterion.

## Deploy plan template

| Item | Question |
|---|---|
| **Target environment** | Where will the change run? |
| **Ordering constraints** | Which resources must be created, updated, or deleted in sequence? |
| **Stateful resources** | Which volumes, databases, or credentials outlast the deployment? |
| **Secret references** | Are secrets named by reference, never by value? |
| **Health check** | What observable signal proves the change is live and correct? |
| **Blast radius** | What breaks if the deploy succeeds partially? |

## Rollback plan template

| Item | Question |
|---|---|
| **Previous known-good state** | Image tag, commit SHA, or manifest version. |
| **Rollback trigger** | Which signals cause an automatic or manual rollback? |
| **Data safety** | Will rollback lose or corrupt user data? |
| **Verification after rollback** | How do we prove the rollback restored the expected state? |

## Worked example: adding a backend to a multi-tenant service

The Builder is asked to add a new memory backend. The change itself is the backend driver and configuration. The deploy plan specifies that the new backend must be seeded before the API is restarted, because the API reads backend availability at startup. The rollback plan names the previous API image tag and the migration that must remain reversible. The Verifier publishes a gate: after deploy, a synthetic tenant write must round-trip through the new backend without 500 errors.

Without the landing plan, the Builder might implement the driver, pass unit tests, and hand off a manifest that restarts the API before seeding. The API would fail to start, and the Runtime would be left guessing whether the problem is the driver, the config, or the order.

## Worked example: an operations runbook change

The Builder is asked to add a new runbook step for a recurring alert. The change is the runbook text and the automation that executes the recovery command. The deploy plan specifies that the runbook must be published to the incident-response tool and the automation must be registered with the on-call scheduler before the next shift starts. The rollback plan names the previous runbook version and the schedule entry to restore. The Verifier publishes a gate: a synthetic alert must trigger the runbook and the automation must complete without manual intervention.

## Why this deserves its own page

Landing planning spans three roles and several lifecycle stages. The main paper introduces the idea in two paragraphs; a standalone page can provide the actual structure, worked examples, and criteria for "ready to land."
