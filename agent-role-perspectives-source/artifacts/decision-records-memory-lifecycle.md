# Decision Records and the Memory Lifecycle

This page expands the Archivist strategy and lesson in *Working Together*: record the decision, not only the result, and the memory lifecycle during and after a task. Fleet memory is not a single store; it is a lifecycle with promotion and pruning rules.

## Two modes of memory

During a task, memory should be cheap, local, and disposable:

- Drafts
- Hypotheses
- Transient context
- Per-agent preferences

After a task, memory must be promoted or pruned:

- Validated decisions
- Reusable conventions
- Failure recoveries
- Runtime observability rules

## Promotion criteria

A memory is promoted to durable, shared storage when:

- It has been validated by the relevant role.
- It is attributed to a source.
- It is written in terms that another agent can act on.
- It answers a question the fleet is likely to ask again.

## Decision-record template

| Field | Purpose |
|---|---|
| **Decision** | What was chosen. |
| **Context** | The problem or question that required the decision. |
| **Alternatives considered** | What was rejected and why. |
| **Reasoning** | Why the chosen path was selected. |
| **Owner** | Which role owns the decision. |
| **What would change the verdict** | Invalidating assumptions or new evidence. |
| **Related artefacts** | Links to code, gates, or runbooks. |

## Pruning rules

A memory should be pruned when:

- It describes a system state that no longer exists.
- It contradicts a newer decision without explaining the change.
- It was a draft that was never validated.
- It duplicates an existing record without adding detail.

## Example: a promoted convention

The fleet discovers that rollback tags must include a timestamp and a role identifier. During the task, the Builder experiments with several tag formats. After the task, the Archivist promotes the chosen format as a fleet convention, records why the other formats failed, and notes that the Verifier will check future tags against this format. The draft formats are discarded.

## Example: an operations recovery record

After a live incident, the Archivist records why the recovery worked, not only that the service recovered. The decision record names the rollback target, the precondition the Runtime verified, the signal that triggered the rollback, and the false-positive that previously delayed the same decision. The next time the same alert fires, the on-call agent does not have to rediscover the recovery path. If the alert turns out to be a different root cause, the decision record is updated with a new invalidating assumption rather than overwritten.

## Why this deserves its own page

The Archivist's lesson is about a lifecycle, not a single practice. A separate page lets the fleet define "promote or prune" as a repeatable process and gives the Archivist a concrete artefact to maintain, similar to how the Builder and Researcher each have their own hand-off formats.
