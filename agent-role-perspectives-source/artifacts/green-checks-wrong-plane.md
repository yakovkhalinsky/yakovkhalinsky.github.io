# Green Checks on the Wrong Plane: Designing End-to-End Gates

This page expands the Verifier anti-pattern in *Working Together*: green checks on the wrong plane. A check that passes at the wrong level of abstraction gives the fleet a false license to advance.

## What "wrong plane" means

A check is on the wrong plane when it verifies an intermediate artefact rather than the user-level outcome. Common mismatches:

| Plane observed | Plane that mattered | Failure |
|---|---|---|
| Unit tests pass | The rendered manifest drops a required field | Service starts but rejects requests |
| Template renders cleanly | The live environment has drifted from the test fixture | Deploy succeeds, runtime fails |
| Mock backend returns expected value | Real backend is unreachable | Integration fails in production |
| Precondition report is present | Precondition report is stale | Verifier judges against ghost state |

## Catalogue of cases

### Silently dropped values

A template engine removes a field that is present in the source schema but absent from the live data model. Every individual test passes because no test exercises the rendered output against the live parser.

### Mocks that hide reality

A test suite uses in-memory implementations for external services. The in-memory versions behave correctly, but the real services have different timeout, auth, or encoding behaviour.

### Stale precondition reports

The Runtime publishes a report saying all services are present. By the time the Verifier evaluates the gate, one service has been restarted with a new identity. The gate passes against a report that no longer describes the environment.

## Designing end-to-end gates

A gate is on the right plane when:

- It exercises the artefact in the shape and context in which it will actually run.
- It observes the live target environment directly, not a cached report.
- It checks user-level outcomes, not only per-step status codes.
- It can fail for a specific, attributable reason.

## Example checklist

- [ ] The artefact is rendered or built exactly as the Runtime will receive it.
- [ ] The live environment is observed, not summarised.
- [ ] At least one check exercises the full path from user input to user output.
- [ ] Failure produces a concrete reason and owner.

## Why this deserves its own page

The paper names the anti-pattern in a few sentences, but the failure mode is rich and recurrent. A dedicated page mirrors the prior fleet-research *Incident Archetypes* artefact and gives the Verifier a field guide they can apply during gate design.
