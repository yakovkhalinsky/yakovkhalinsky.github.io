# Evidence Packets: A Research Hand-off Format

This page expands the Researcher strategy in *Working Together*: evidence packets with a recommendation and caveats. The evidence packet is the standardised hand-off from Researcher to Dispatcher or Builder. Its purpose is to turn exploration into a framed decision.

## Packet template

| Field | Purpose |
|---|---|
| **Question asked** | The exact decision the packet is meant to inform. |
| **Assumptions** | What the Researcher believes is true and did not verify. |
| **Sources** | URLs, documents, commands, or observations consulted. |
| **Alternatives considered** | At least two, with effort and risk notes. |
| **Confidence ratings** | High / medium / low for each claim. |
| **Recommended path** | One clear option, with a one-sentence rationale. |
| **Caveats** | What could make the recommendation wrong. |
| **Invalidating assumptions** | What would change the verdict. |

## Strong example

> **Question:** Which sync contract should the fleet memory layer use?  
> **Assumptions:** Edges may be offline for minutes to hours; conflicts are rare but must be preserved.  
> **Sources:** Surveyed six sync strategies and a live probe of two candidate tools.  
> **Alternatives:** (A) primary–replica replication — simple but blocks offline writes; (B) CRDTs — offline-friendly but complex merge semantics; (C) append-only oplog — matches the append-mostly workload and preserves provenance.  
> **Confidence:** High that oplog is the natural default; medium that CRDTs would work for specific counter-like data.  
> **Recommendation:** Adopt append-only oplog with vector-clock watermarks as the default.  
> **Caveats:** Requires a compaction job; vector clocks must be assigned per role.  
> **Invalidating assumptions:** If the workload becomes write-heavy or the fleet requires strong query consistency, re-evaluate primary–replica or consensus.

## Strong example: an operations scenario

> **Question:** Is the current alert spike caused by the new deployment or by a downstream partner change?  
> **Assumptions:** The monitoring pipeline is not dropping alerts; downstream partner status is independently observable.  
> **Sources:** Last four hours of alerts, deploy log, and partner status page.  
> **Alternatives:** (A) Roll back the deployment immediately; (B) wait and correlate with partner status; (C) roll back only the affected service region while observing the partner channel.  
> **Confidence:** Medium that the partner change is the dominant cause; low that a full rollback is necessary.  
> **Recommendation:** Roll back the affected region first, keep the others live, and compare alert rates before deciding on a global rollback.  
> **Caveats:** If the partner does not recover within fifteen minutes, the deployment may still need a global rollback.  
> **Invalidating assumptions:** If alerts also appear in regions with no dependency on the partner, the deployment is the primary cause.

## Weak example

> "I looked at a few options. Option A seems good. Here are some links."

This is weak because it contains no alternatives, no confidence, no caveats, and no invalidating assumptions. The Builder receiving it will either re-research or build on unstable ground.

## Completion criteria

An evidence packet is complete enough to hand off when:

- The question is narrower than the original prompt.
- At least two viable alternatives are named.
- Each alternative has a concrete trade-off.
- The recommendation includes a reason and a caveat.
- The Dispatcher or Builder can act without asking for more options.

## Why this deserves its own page

The main paper introduces evidence packets in a single paragraph. In a running fleet, they are a reusable artefact that shapes every Researcher→Builder hand-off. A dedicated page turns the idea into a format the fleet can consistently produce, review, and improve.
