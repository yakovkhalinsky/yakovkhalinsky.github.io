# Designing a Local-First Memory Runtime for Agent Fleets

**Abstract.** We examine why long-running multi-agent fleets often abandon shared memory tools, and what design patterns make memory usable again. Drawing on a six-role operations fleet, we identified three recurring anti-patterns — the blocking-embedding path, the heavy default stack, and remote-only storage — and three counter-patterns: full-text-first retrieval, asynchronous embedding queues, and a local-first runtime whose fleet mode is a configuration change rather than a different product. We report on the operational gains and losses observed when these patterns were adopted, and on the contracts that remain fragile: single-writer concurrency, tombstone retention, and hub-identity pinning. The paper is intended as an experience report for builders who want durable agent memory without presuming a data-centre footprint.

## 1. Setting and motivation

We operated a six-role fleet — Dispatcher, Builder, Runtime, Verifier, Researcher, and Archivist — for long enough that continuity of context mattered more than any single task. A Builder needed to know why a previous refactor had been rejected. A Runtime agent needed to remember the last known-good configuration. The Archivist needed to keep skills, runbooks, and incident lessons consistent across sessions and across agents. Memory was not a convenience; it was the coordination surface.

Yet the shared memory layer was failing in a quiet way: agents were not using it. They kept context in prompt prefixes, local files, or ad-hoc notes rather than in the shared memory surface. The surface offered vector search, hybrid retrieval, session scoping, and a role-aware data model. The reason for avoidance was friction. Every memory write blocked on a remote embedding call. Every recall required a round trip through a network gateway, a vector database, and a relational database. Running the stack locally meant orchestrating several services and a container orchestrator before a single fact could be stored.

We set out to answer a narrower question: what is the smallest durable memory runtime that a single agent or a small fleet can actually rely on?

## 2. Anti-pattern: the heavy default stack

A common assumption in cloud-native agent memory is that the default deployment should mirror a production backend: a relational store, a vector store, a separate embedding service, an in-memory coordination service, a network gateway, and an orchestrator. Each component is reasonable on its own; together they form a heavy default stack for what is, at small scale, mostly append-only text, messages, and conclusions.

The costs we observed were concrete. A developer who wanted memory on a laptop had to stand up several services before the first fact could be stored. Recovery meant restoring multiple stateful systems in the right order. More importantly, the operational weight changed behaviour: agents kept notes in local scratchpads because the shared path was too slow and too brittle for routine use. The memory layer existed, was technically capable, and was effectively unused for cross-session work.

We treat this as an instance of the broader anti-pattern **tool success, mission failure**: every component works, but the composed workflow fails its real purpose because the cost of using it exceeds the benefit.

## 3. Anti-pattern: the blocking-embedding path

The second anti-pattern is coupling the memory write path to embedding inference. If every remember-style call blocks until an embedding service returns a vector, the whole memory surface stalls when the embedder is slow, busy, or unreachable. Agents learn to keep facts in prompt context or local scratchpads. Those scratchpads are fast, but they vanish on restart or hand-off.

We observed this directly. During inference spikes, write latency rose from milliseconds to several seconds. The shared store became a cold archive for data written during quiet periods, not a living memory used in real decisions. The symptom was not a crash but a gradual desertion of the memory surface.

## 4. Pattern: local-first runtime with full-text-first retrieval and asynchronous embedding

The counter-pattern we found effective is a deliberately minimal edge runtime: one process, a local file-backed database, full-text search as the default retrieval path, asynchronous embedding, and optional federation through a shared hub that is enabled by configuration rather than by a different product.

### 4.1 Single edge runtime

The pattern is a single runtime package that bundles the API surface, the agent protocol, the local store, and optional federation. In local mode the package runs as one binary or container, binds to a local-only address, and needs no external network. In fleet mode the same package is pointed at a shared hub by a configuration change; the agent-facing surface does not change.

This is not a new architectural idea. The novelty, if there is one, is applying it to agent memory rather than to a web application. The single package gives the local agent a stable, inspectable data file that can be read, backed up, and restored with ordinary tools. When something goes wrong, an operator can inspect the local store directly. That debuggability turned out to be more useful than any single advanced retrieval feature.

### 4.2 Single-file local store with embedded search

The minimal local store is one durable file — for example a single-file relational database with transaction logging — that holds peers, sessions, messages, conclusions, session metadata, and a sync watermark. A full-text index and a small vector-search extension are loaded inside the same file. A separate vector server is unnecessary at small scale.

This pattern was driven by the observation that agent memory at the edge is mostly read-heavy, append-mostly, and small enough to fit on a laptop disk. A single file also makes backup trivial: stop the edge runtime, archive the database and its transaction log, and store the archive. Restore is the reverse.

### 4.3 Full-text-first retrieval

Retrieval defaults to full-text search, not vector similarity. Agents get answers immediately because full-text search requires no model inference. Vector search is available, but it is a secondary, not blocking, path. A memory write is indexed for text search as soon as it is committed; the embedding is queued and applied asynchronously.

This inverts the common default. Previously, embedding was the primary path and text search an add-on. The local-first default treats text as the reliable path and vectors as the richer but slower path. In practice, many agent recalls are satisfied by text search, and the fleet does not stall when the embedding queue is behind or unavailable.

### 4.4 Asynchronous embedding

The pattern is a durable embedding queue inside the same runtime. A write commits the text and metadata immediately and records an indexing job; a background worker later computes vectors using a local or remote embedding provider. Once the vector is ready, the row is updated.

The queue is not in memory. Jobs are stored durably in the same file-backed database and acknowledged only after the update is committed. A crash loses no queued work because the queue is the database. This addresses the recurring fear that an asynchronous queue would silently drop embeddings during a restart.

### 4.5 Federation as a configuration change

When the operator points the edge runtime at a fleet hub, the federation daemon wakes up. The pattern is to maintain an append-only operation log — an oplog — in the local database. Each write appends a frame describing what changed, who changed it, and a vector-clock watermark. The daemon pushes pending frames to the hub and pulls missing frames on reconnect; the transport merely has to authenticate the hub. The hub stores the authoritative log and lets other runtimes replay it.

The sync protocol is intentionally simple. The domain is mostly append-only: new messages, new conclusions, new sessions. Conflicts are rare and, when they occur, are recorded for review rather than silently resolved by last-write-wins. The watermark is a vector clock, not a wall-clock timestamp, so concurrent writes from different edges can be ordered without pretending that clock synchronisation is reliable.

Importantly, the local runtime remains the agent's source of truth for reads. Fleet mode adds durability and aggregation; it does not turn the local store into a cache that must phone home on every recall.

### 4.6 Explicit identity and local-first promotion

One lesson from earlier shared memory tools was that default identity buckets caused silent misrouting. Agents wrote memories that other agents could not see because the defaults silently partitioned the data. The effective surface requires explicit caller and subject identity on every memory operation. There is no generic bucket.

We also distinguished local memory from shared memory. A local memory is draft, private, or transient. A shared memory is a fact that has been verified and attributed. Promotion from local to shared is explicit, not automatic. This reduced noise in the shared store and prevented half-formed reasoning from leaking across agents.

## 5. Operational trade-offs

### 5.1 Gains

**Reduced footprint.** A personal agent can run with a single binary and a data file. Setup time drops from orchestrating multiple services to starting one process.

**Immediate writes and recalls.** Because the write path no longer blocks on embedding, agents use memory for small, frequent notes. Because full-text search answers first, recalls return in milliseconds.

**Simpler backup and restore.** A local archive contains the database, configuration, sync watermark, and model cache. Restore is a single command. Fleet recovery is either a local restore followed by sync, or a fresh edge reseeding from the hub.

**Inspectability.** Operators can query the local database with ordinary tools. This made debugging faster than tracing through a distributed stack.

**Mode transparency.** The agent-facing API and agent-protocol surface is the same in local and fleet mode. An agent written against the local runtime graduates to a fleet without code changes.

### 5.2 Losses and new constraints

**Single-writer concurrency.** A file-backed database with transaction logging is still a single writer. Multiple edge runtimes cannot safely share the same file. For a personal fleet this is a natural constraint; for larger edge deployments it requires routing all writes for a workspace through one runtime process or a write queue.

**Brute-force vector limits.** The local vector extension works well for thousands of memories but will not scale to millions. A fleet that outgrows the local extension can point the hub at a dedicated vector database while keeping the same agent surface.

**Sync complexity is real but deferred.** The simple oplog works because the domain is append-mostly. It still requires idempotent pushes, monotonic watermark reads, tombstone retention, conflict logging, and pinned hub identity. These are not free; they are simply smaller and more contained than a backend stack that couples the same concerns to separate services.

**Embedding lag.** Asynchronous embedding means a recalled memory may not yet be vector-searchable. The design accepts this by making full-text search the default and by treating vector search as a quality improvement, not a correctness gate.

## 6. Lessons for builders

Several lessons from this experience generalise beyond the specific stack.

**Optimise the path agents actually use.** A memory layer that agents avoid is worse than a simpler one they use constantly. Reducing the cost of the common case — a quick note and a quick recall — mattered more than supporting every advanced retrieval mode.

**Make local mode truly local.** If local mode still requires a remote service to function, it is not local mode. The test is whether an agent can remember and recall with no network and no dependencies beyond the runtime.

**Default to text, not vectors.** Full-text search is fast, deterministic, and requires no model. It is a better default for most agent memory lookups. Vector search should be layered on top, not assumed as the primary path.

**Queue embeddings durably, not in memory.** An in-memory queue creates a durability gap that is hard to explain to operators. Storing the queue in the same durable store as the data removes an entire class of crash-loss incidents.

**Require explicit identity from day one.** Shared memory tools that default to a generic caller or subject create partitions that agents cannot see. Reject ambiguous calls rather than silently picking a bucket.

**Distinguish local draft from shared fact.** Automatic promotion of every agent thought into shared memory pollutes the shared store. Promotion should be explicit and reversible.

**Do not use a message bus as the source of truth.** We briefly considered using a pub/sub service for sync notifications, then rejected it because the delivery model is at-most-once. A durable operation log with pull-based catch-up is a slower but safer foundation. Pub/sub can be used as a hint layer, not as a correctness layer.

**Keep the agent surface stable across modes.** One of the more useful properties of the local-first pattern is that local and fleet mode expose the same API and agent-protocol tools. Agents should not need rewrites when the deployment grows.

## 7. Threats to validity

This report has the usual limits of a single-system field study. We observed, shaped, and operated the edge runtime over time, so observer effects and hindsight bias are likely. The workload was dominated by operational tasks — infrastructure, code, documentation, and incident response — and may not match fleets doing creative, scientific, or customer-facing work.

We also cannot claim that the local-first approach is always cheaper. The trade-offs flip once the fleet grows large enough that single-writer file-backed databases, brute-force vector search, or per-edge storage become bottlenecks. For such fleets, the value of this pattern is as a small-fleet default and a clear upgrade path, not as a large-scale architecture.

Finally, the reported gains in agent adoption are observational, not measured by a controlled experiment. Agents began using memory more often, but we did not instrument a randomised comparison. Future work could quantify recall success rates, time-to-recall, and memory-write frequency before and after adopting these patterns.

## 8. Conclusion

We examined why a long-running multi-agent fleet abandoned its shared memory tools and what patterns restored their usefulness. The local-first runtime pattern — file-backed storage, full-text-first retrieval, asynchronous embedding, and optional federation through an append-only operation log — trades advanced per-component capability for a smaller, more reliable common path. The agents in the fleet began using memory once the cost of using it dropped below the cost of avoiding it.

The deeper lesson is that agent memory infrastructure should be judged by whether agents trust it for the small, frequent, cross-session facts that hold coordination together. Trust comes from low latency, durability, and debuggability more than from feature richness. We offer this field study to builders who are facing the same tension between capability and operability.

## References

See [fleet-memory bibliography](../artifacts/fleet-memory-bibliography.md).
