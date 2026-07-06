# Fleet Memory Incident Archetypes

Anonymised incident archetypes drawn from a redesign of a long-running multi-agent fleet's memory layer. These archetypes are additional to the general incident archetypes catalog and are intended to support the fleet-memory paper set.

## 1. The embedded-replica trap

**Decision tension:** Whether to reuse the local database's native replication mechanism as the durable sync protocol, or to define a separate application-level operation log.

**Root cause:** A storage backend is attractive because it ships replication primitives out-of-the-box. However, those primitives may require the application to quiesce local reads during sync, and they force operators to understand two sync abstractions at once.

**Manifestation:** The backend vendor's documentation warns that opening the local database while the embedded replica is syncing can corrupt data. A long-running gateway cannot pause all reads whenever sync runs.

**Lesson:** The sync wire protocol should be decoupled from the local storage engine. Treat the backend as a persistence target, not as the transport. An application-level append-only operation log is the single sync abstraction that operators debug, back up, and restore.

**Research question:** Does the paper conflate storage replication with application sync? Does it explain what happens when local reads and sync overlap?

## 2. The notification-as-source-of-truth

**Decision tension:** Whether to use an at-most-once pub/sub channel as a durable sync transport or as a transient hint layer.

**Root cause:** Pub/sub systems are fast and convenient for real-time notifications, but they are fire-and-forget. Any gateway offline during a fleet update will miss events permanently.

**Manifestation:** A previous design used an in-memory coordination service to broadcast fleet events. A gateway that rebooted during a broadcast missed the update permanently.

**Lesson:** Pub/sub can cache-invalidate or hint, but the durable source of truth must remain the database or operation log. A pull-based reconciliation loop must recover missed events on reconnect.

**Research question:** Does the paper distinguish notification latency from sync correctness? Does it evaluate behaviour after an offline or missed-message window?

## 3. The blocking-embedding path

**Decision tension:** Whether to compute embeddings synchronously at write time or to queue them asynchronously and index text immediately.

**Root cause:** Agents avoid memory tools that block on remote or local embedding inference. If every remember call must wait for a vector model, the tool becomes unusable in interactive contexts.

**Manifestation:** The previous low-level tools were avoided precisely because synchronous embedding made writes slow.

**Lesson:** Separate search availability from embedding freshness. Full-text search gives instant recall; vector search becomes available once embeddings complete. The write path must never wait on the embedder.

**Research question:** Does the paper report end-to-end write latency as well as embedding latency? Does it measure whether agents actually use the memory tool under each regime?

## 4. The anonymous-agent default

**Decision tension:** Whether memory tools should carry sensible-looking default IDs or require explicit actor identity.

**Root cause:** Defaults reduce friction for quick demos, but they silently merge memories across agents and users, breaking provenance and creating cross-contamination.

**Manifestation:** Existing tools defaulted to generic agent and user IDs. The comprehensive review flagged this as dangerous; the new surface requires explicit caller and subject identity for every call.

**Lesson:** Identity and provenance are not ergonomic details. Every memory write must encode the actor, the observed peer, the workspace, and the session. Defaults should be removed, not hidden.

**Research question:** Does the paper evaluate identity hygiene? Does it measure how often default IDs cause retrieval errors or privacy violations in multi-agent settings?

## 5. The scale-stack default

**Decision tension:** Whether a personal/small-fleet gateway should default to a heavyweight database, vector store, orchestrator, and observability stack, or whether those should be opt-in.

**Root cause:** Enterprise-grade components are justified at fleet scale but become barriers for a single user on a laptop or edge device.

**Manifestation:** The target architecture made the relational backend, vector backend, dedicated embedding server, in-memory cache, container orchestrator, and full observability stack optional. The first milestone starts a single binary with no external network dependencies.

**Lesson:** Scale is opt-in. The default runtime must prove value on modest hardware before heavier services are added. Research claims about fleet memory must distinguish personal-fleet from enterprise-fleet contexts.

**Research question:** Does the paper specify the deployment mode? Are optional services clearly separated from mandatory correctness dependencies?

## 6. The single-transport assumption

**Decision tension:** Whether to hard-code a single network overlay or to remain transport-agnostic.

**Root cause:** A particular overlay is convenient, but it is not universal. Operators may prefer direct private IPs, WireGuard, SSH tunnels, mTLS over the public internet, or local sockets.

**Manifestation:** Early wording implied the overlay was the only exposure path. The final architecture treats the overlay as optional convenience and supports any secure transport with a plain HTTPS hub URL.

**Lesson:** Networking is a deployment choice, not an architectural invariant. Fleet memory sync should run over any transport that provides TLS and identity.

**Research question:** Does the paper evaluate across at least two transports? Does it avoid conflating the overlay's identity model with the application's authorisation model?

## 7. The conflict-resolution semantics gap

**Decision tension:** Whether to use last-write-wins, a full CRDT, or an append-only operation log with vector-clock causality and explicit supersession.

**Root cause:** Last-write-wins loses agent intent for session and peer-card updates. Full CRDTs are expensive and semantically awkward for non-commutative operations such as messages.

**Manifestation:** The team rejected both naive primary-replica and full CRDT approaches. The chosen design preserves concurrent writes, uses vector-clock watermarks for ordering, and lets intentional supersession declare when one fact replaces another.

**Lesson:** Match the conflict policy to the data semantics. Append-mostly memory data needs causality tracking and a visible conflict log, not automatic merge.

**Research question:** Does the paper define the conflict model per entity type? Does it expose conflicts for operator or agent review, or does it silently discard divergent writes?

## 8. The implicit-promotion boundary

**Decision tension:** Whether local agent memories automatically become fleet memories, or whether promotion is explicit and deduplicated.

**Root cause:** Local-first memory tools can accumulate large volumes of personal, transient, or draft notes. Auto-sharing everything to the fleet creates noise and privacy risk.

**Manifestation:** The API designer proposed explicit promotion tools and an optional scheduled review loop that scans shareable local entries and deduplicates by content hash before writing to the fleet.

**Lesson:** Local memory and fleet memory are different trust and visibility scopes. Promotion must be intentional and idempotent, not automatic.

**Research question:** Does the paper define the boundary between local and shared memory? Does it measure recall quality under automatic vs. explicit sharing?

## 9. The single-writer backend bottleneck

**Decision tension:** Whether a single-process, file-backed database can serve concurrent agents, or whether a write queue / single writer discipline is required.

**Root cause.** A file-backed database allows concurrent reads, but multiple processes writing the same database file can deadlock or corrupt.

**Manifestation.** The runtime engineer flagged single-writer file-backed concurrency as a high-severity risk. The architecture funnels writes through one gateway process with an internal queue.

**Lesson:** A local-first backend does not automatically provide multi-writer concurrency. The runtime must serialise writes or shard by writer.

**Research question:** Does the paper report throughput and contention under concurrent agents? Does it define the writer-serialisation mechanism?

## 10. The convenience-first security default

**Decision tension:** Whether to optimise for zero-config local operation or fail-closed security in any remote mode.

**Root cause:** A system that binds to localhost with no authentication is convenient locally, but accidentally exposing the same configuration to a network interface creates an open memory store.

**Manifestation:** The existing stack had a documented weak-default signing secret. The new design requires operator-set secrets before fleet mode and refuses anonymous remote binds.

**Lesson.** Local mode can be permissive; remote mode must be fail-closed. Default binds should be local-only, and any non-local bind should require explicit authorisation configuration.

**Research question:** Does the paper evaluate security defaults? Does it test that remote mode rejects anonymous requests and that credentials are not logged?

## 11. The garbage-collection window race

**Decision tension:** How long to retain tombstones and old operation-log frames before garbage collection.

**Root cause:** Tombstones are needed to propagate deletes to offline edges, but retaining them forever grows storage indefinitely.

**Manifestation:** The ops reality-check recommended a 30-day tombstone retention default, a documented reseed path, and snapshot backups before GC.

**Lesson:** Offline capability is bounded by the tombstone / operation-log retention window. That bound must be explicit, and a reseed or snapshot path must exist for edges that exceed it.

**Research question:** Does the paper state the retention window and the recovery path for long-offline edges? Does it measure data loss vs. storage cost trade-offs?

## 12. The hub-identity ambiguity

**Decision tension:** Whether an edge trusts any reachable hub URL or pins the hub's cryptographic identity.

**Root cause:** An edge that reconnects to a different hub instance may replay or accept stale state.

**Manifestation:** The ops reality-check recommended pinning hub identity via certificate or CA fingerprint in edge configuration.

**Lesson:** Sync watermarks only make sense with respect to a stable hub identity. The runtime must authenticate the hub, not only the edge.

**Research question:** Does the paper address hub identity and split-brain recovery? Does it evaluate behaviour when an edge reconnects to a different hub instance?
