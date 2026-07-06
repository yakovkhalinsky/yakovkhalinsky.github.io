# Failure Modes in Federated Agent Memory

**Abstract.** We report on the failure modes that emerge when agent memory moves from a single process to a federated edge-and-hub architecture. In this shift, the common failure class changes from local code bugs to distributed contract failures: durability, identity, ordering, retention, and authority contracts that were implicit in the single-process design become visible only when they break. We describe six incident archetypes observed in a long-running six-role operations fleet: notification treated as a source of truth, the embedded-replica trap, the blocking-embedding path, the anonymous-agent default, the tombstone garbage-collection race, and hub-identity ambiguity. For each archetype we give the interaction pattern, an anonymised example, and the contract or ritual that contained it. The paper also connects each archetype to the patterns that contain it and to the broader reliability-engineering literature. It is intended as an incident experience report for practitioners building federated agent memory and as a companion to the sync taxonomy and memory-centric position papers.

## 1. The shift from code bugs to contract failures in federated memory

When memory lives inside one agent or one process, a memory failure is usually a local bug: a wrong retrieval, a stale cache, a bad summarisation. The fix is local too — correct the function, refresh the cache, tighten the prompt. When memory is federated across edges and a shared hub, the picture changes. Each edge can be correct in isolation while the fleet as a whole fails, because the contract between the parts is wrong or missing.

We operated a fleet organised around six stable roles — Dispatcher, Builder, Runtime, Verifier, Researcher, and Archivist — over an extended period. As memory responsibilities spread across edges and a hub, the incidents that consumed considerable time were rarely caused by a single bad tool call. They were contract failures. A durability contract failed: an at-most-once notification was treated as a durable update. A storage contract failed: an embedded replication mechanism was used as the sync protocol. A latency contract failed: every write blocked on embedding inference. An identity contract failed: generic defaults merged memories across agents. A retention contract failed: tombstones were collected before an offline edge returned. An authority contract failed: an edge accepted state from a hub it could not authenticate.

These failures are harder to debug than local bugs because every individual component can pass its own tests. The symptom often appears hours or days after the breach, in a different role, with no stack trace pointing back to the contract. The lesson we took from this is that federated memory must be designed as a set of explicit contracts first, and as an implementation second. The rest of this paper describes the six archetypes that taught us that lesson.

## 2. Archetype 1: notification-as-source-of-truth

**Pattern.** A fast, at-most-once publish/subscribe channel is used to distribute memory updates across the fleet. Because notifications arrive quickly, the team assumes they are also durable.

**Root cause.** Notification latency is mistaken for sync correctness. A channel that drops messages when a subscriber is offline is treated as the source of truth rather than as a hint.

**Manifestation.** A previous design used an in-memory coordination service to broadcast fleet events. The channel was fast and simple, but it was fire-and-forget. A gateway that rebooted during a broadcast missed the update permanently. When it rejoined, its local store was consistent with itself but divergent from the rest of the fleet. The divergence was not visible until a downstream agent made a decision based on stale memory. The problem was compounded by the fact that the notification path had no delivery acknowledgement, so the team had no signal that a node had missed frames.

**Lesson.** Durable sync must be backed by an append-only operation log, not by a notification channel. Notifications are useful for cache invalidation or as a hint that new state is available, but the safe recovery path is a pull-based reconciliation loop that reads the log from a known watermark. This maps to the *notification-as-source-of-truth* archetype in the site catalogue and is the counter-pattern to the append-only operation log and push/pull sync agent patterns. Adding a sync-status surface that reported pending frames and last successful pull made the divergence visible before downstream agents could act on stale memory.

## 3. Archetype 2: embedded-replica trap

**Pattern.** A storage engine that offers built-in replication is adopted as the default sync transport. The application reads and writes through the same engine, so replication appears to come for free.

**Root cause.** Storage replication and application sync are conflated. The engine's frame-level replication becomes a second, hidden sync abstraction that operators must understand alongside the data model.

**Manifestation.** A Researcher prototyped a fleet gateway using an embedded replica engine. The prototype worked in short demos, but the engine's own documentation warned that opening the local database while the replica was syncing could corrupt data. A long-running gateway cannot pause all local reads during every sync window. The team rejected the approach as the default because it forced operators to reason about two sync layers: the engine's replication frames and the application's operation log.

**Lesson.** Decouple the sync wire protocol from the local storage engine. Treat the local database as a persistence target, not as the transport. A single application-level operation log is the sync abstraction that operators debug, back up, and restore. This is the *embedded-replica trap* archetype; the counter-pattern is to keep sync explicit and storage pluggable, as described in the sync strategy taxonomy. The decision allowed us to keep the local storage engine lightweight while changing the sync transport between local sockets, a private IP, and an encrypted overlay without touching the data model.

## 4. Archetype 3: blocking-embedding path

**Pattern.** Every memory write waits for a vector embedding before it returns. Vector search is treated as the primary or only retrieval mode.

**Root cause.** The write path is coupled to inference latency. When the embedder is slow, busy, or offline, the whole memory surface stalls.

**Manifestation.** In an earlier backend, every remember-style call had to wait for a remote embedding service before the write could commit. During inference spikes, write latency rose from milliseconds to several seconds. Agents stopped using the memory tool and kept facts in local scratchpads or prompt context. Those scratchpads were lost on restart, and the shared store was starved of the facts it was supposed to hold. The symptom was not a crash but a gradual desertion of the memory surface.

**Lesson.** Separate search availability from embedding freshness. Default to deterministic full-text search, which returns in milliseconds, and queue semantic embedding asynchronously. Vector recall becomes available once embeddings complete, but the write path never waits on the embedder. This is the *blocking-embedding path* archetype; the counter-patterns are the full-text-first recall surface and the asynchronous embedding queue. The practical result was that agents resumed using the memory tool for routine facts; richer semantic search remained available for harder queries once the queue drained.

## 5. Archetype 4: anonymous-agent default

**Pattern.** Memory tools fall back to a generic identity when the caller does not supply one. The default looks sensible in a demo but is dangerous in a fleet.

**Root cause.** Defaults encode a single-agent, single-user assumption. In a multi-agent fleet, they silently merge provenance across agents and users.

**Manifestation.** Early memory tools defaulted to a generic agent and user identity. One Runtime agent wrote operational notes into the default bucket. Later, a different agent queried with its own explicit identity and found nothing. Downstream agents then regenerated credentials because they believed the notes did not exist. The failure was not a retrieval bug; it was an identity contract violation.

**Lesson.** Reject ambiguous calls rather than silently defaulting to a shared bucket. Every memory write must carry explicit caller identity and subject identity, plus workspace and session context. This is the *anonymous-agent default* archetype, closely related to the *silent dependency break* archetype in the catalogue; the counter-pattern is explicit identity in shared tools. The Verifier added a regression check that fails any memory write lacking explicit caller and subject identity, so the contract is checked on every build rather than enforced only by code review.

## 6. Archetype 5: tombstone garbage-collection race

**Pattern.** Deleted facts are represented by tombstone records, but those tombstones are garbage-collected after a fixed retention window. An edge offline longer than the window misses the delete.

**Root cause.** The trade-off between storage cost and offline capability is left implicit. There is no documented recovery path for edges that exceed the retention window.

**Manifestation.** The fleet used tombstone records to propagate deletes. To control disk growth, a short retention window was chosen. One edge was offline for maintenance longer than the window. When it reconnected, it pulled only the live operation-log frames and never saw the tombstone. A deleted conclusion resurrected on that edge and influenced a deployment decision. The other nodes had already purged the fact.

**Lesson.** Make the retention window explicit and bound offline capability by it. Before garbage collection, snapshot the log. Provide a reseed path for edges that return after the window. Expose sync status so operators can see which edges are lagging. This is the *garbage-collection race* archetype; the counter-pattern is tombstone soft deletes with a documented retention and recovery policy. We found that a 30-day window was long enough for personal edges that are occasionally offline and short enough to keep storage growth bounded, provided the trade-off was explicitly documented so operators knew what they were choosing.

## 7. Archetype 6: hub-identity ambiguity

**Pattern.** An edge trusts any reachable hub address. A load-balancer or DNS change can point the edge at a different hub instance with older state.

**Root cause.** Sync watermarks are treated as absolute, but they are only meaningful with respect to a stable hub identity. The edge authenticates itself to the hub, but not the hub to the edge.

**Manifestation.** An edge synced to a hub behind a load balancer. A configuration change shifted traffic to a freshly restored hub replica that held an older operation log. The edge pulled from the new hub, accepted the older state, and replayed deletes and updates in the wrong order. The fleet briefly held two authoritative views until the mismatch was detected by a Verifier audit. The trigger was not a compromise; it was a routine DNS change combined with a missing identity check.

**Lesson.** Pin hub identity in edge configuration, using a stable credential or certificate fingerprint, and validate it on every sync session. Watermarks and conflict resolution only make sense against a known hub. This is the *hub-identity ambiguity* archetype; the counter-pattern is authenticated, stable hub identity combined with monotonic watermark reads. The cost of the extra configuration is small compared with the cost of a split-brain recovery, which required a Verifier audit and a manual reseed of the affected edge.

## 8. Recovery and observability lessons

The six archetypes share a common theme: federated memory fails when a contract is implicit. The containment strategies all make the contract explicit and observable.

The first contract is **durability is local first**. Every edge should be able to write to a local append-only log without the hub. When connectivity returns, the edge pushes pending frames. If the local store is corrupted, the edge can reseed from the hub or restore from a snapshot. Local-first durability also means that an offline agent can continue to operate; the federated system degrades gracefully rather than stopping the world.

The second contract is **sync must be idempotent**. Each operation-log frame needs a deterministic key so retries after timeouts do not create duplicates. This turns network retry from a hazard into a normal recovery mechanism. Idempotency is especially important at the boundary between an optimistic local write and the eventual push to the hub, where retries are common.

Third, **watermarks must be monotonic and bound to a stable identity**. An edge resumes from its last known watermark, but only against a hub whose identity it can verify. Without hub identity pinning, watermarks become dangerous. Monotonic reads also prevent an edge from applying a pull batch with gaps, which would otherwise let an older frame slip past a newer one and create ordering violations.

Fourth, **tombstone retention must be a first-class policy**. The operator should choose the maximum offline duration the fleet supports, and the system should enforce that bound. Edges that exceed it reseed rather than resurrect stale data.

Fifth, **observability must expose sync state**. A health surface or status tool should report mode, last successful sync, pending frames, and the identity of the connected hub. Roles need to know whether they are caught up before they act on shared memory.

Finally, **recovery must be operator-simple**. The lengthiest incidents happened when recovery required manual intervention across multiple storage layers. A federated memory system should recover by replaying the operation log or by restoring a snapshot. If recovery requires editing internals, the contract has already failed. Operator simplicity is not a convenience; it is a safety property, because recovery usually happens under pressure, when mistakes are most likely.

These lessons echo the broader SRE emphasis on error budgets, post-mortems, and explicit contracts: the work of Allspaw, Dekker, and Beyer and colleagues on reliability engineering, and Nygard's stability patterns, all apply to federated memory once we treat it as a distributed system rather than a model feature. The companion evaluation-protocol paper turns several of these archetypes into testable acceptance gates.

A note on scope. This paper is a qualitative experience report from one fleet. We do not claim that every federated memory system will encounter all six archetypes, only that the contract failures described here are plausible wherever edges and hubs share mutable state. The patterns that contain them — append-only log, explicit identity, full-text-first retrieval, durable embedding queue, tombstone retention with reseed, and pinned hub identity — are offered as a starting checklist rather than as a general recipe.

## References

See [fleet-memory bibliography](../artifacts/fleet-memory-bibliography.md).

See also the site artifacts: [fleet-memory incident archetypes](../artifacts/fleet-memory-archetypes.md), [fleet-memory patterns catalogue](../artifacts/fleet-memory-patterns.md), [sync taxonomy](06-sync-taxonomy.md), and [explicit identity and async semantics](08-explicit-identity.md).
