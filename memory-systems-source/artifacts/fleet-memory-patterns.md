# Fleet Memory Patterns Catalogue

A catalogue of patterns and anti-patterns observed while designing a local-first, federated memory system for a long-running multi-agent operations fleet. These are kept separate from the general patterns catalogue so readers can treat the fleet-memory paper set as an additional, self-contained contribution.

Each entry uses stylised roles (Dispatcher, Builder, Runtime, Verifier, Researcher, Archivist) and anonymised examples. No product names, schemas, tool names, hosts, or deployment identifiers appear.

## 1. Patterns for agent-facing memory

### 1.1 Full-text-first retrieval surface

**Statement:** Expose full-text search as the default recall mode and make vector or hybrid search opt-in.

**Why it helps:** Reads return immediately; agents can reason about results while embeddings are produced asynchronously.

**When to use:** When agents need fast, deterministic recall and embedding latency would otherwise block the read path.

**Example.** The Builder changed the default recall mode to text search. The local agent found facts in tens of milliseconds, whereas the previous default had to wait for the embedding runtime.

### 1.2 Asynchronous embedding queue

**Statement:** Persist embeddings in a durable background queue so memory writes never block on inference.

**Why it helps:** Write latency stays low; failed embeddings can retry without agent intervention; recall degrades gracefully to text search while embeddings are pending.

**When to use:** Any time the embedding model is local or remote and may be slow, CPU-bound, or flaky.

**Example.** The Builder routed every remember operation through a durable queue. The Runtime observed that CPU spikes from the local inference runtime no longer stalled the agent protocol.

### 1.3 Explicit caller and subject identity

**Statement:** Require fully-qualified agent and user identifiers on every memory operation; reject generic defaults.

**Why it helps:** Prevents silent cross-contamination of memories and makes audit and conflict resolution possible.

**When to use:** When multiple agents share a memory store and attribution, provenance, or conflict resolution matters.

**Example.** The Verifier rejected the default identity pair that every caller had inherited. The new contract forced the Dispatcher to supply explicit agent and user IDs before any fact was stored.

### 1.4 High-level ergonomic tool family

**Statement:** Provide one ergonomic tool family for everyday remember/recall/search/sync-status operations while keeping low-level primitives stable.

**Why it helps:** Reduces cognitive load; migration can happen gradually without breaking existing integrations.

**When to use:** When existing primitives are too granular and agents avoid them.

**Example.** The Builder introduced a small set of ergonomic wrappers for the most common operations. Veteran skills continued to use low-level tools while new skills adopted the simplified surface.

### 1.5 Single local runtime package

**Statement:** Bundle the API, agent protocol, relational store, vector extension, async embedder, and sync agent in one binary or container.

**Why it helps:** Operators can start the gateway with one command; fleet mode becomes a configuration change rather than a new deployment shape.

**When to use:** For local-first deployment where setup must be minimal.

**Example.** The Runtime shipped one gateway package. A researcher ran it on a laptop with no orchestrator and later pointed it at a fleet hub by editing two config values.

## 2. Patterns for local-first fleet sync

### 2.1 Append-only operation log

**Statement:** Record every write as an immutable frame containing payload, actor, and causal metadata.

**Why it helps:** Sync, backup, and reseed reduce to "replay frames"; there is no hidden replication state to reason about.

**When to use:** When data is mostly append-only and you need a simple replay, recovery, and audit story.

**Example.** The Runtime stored each conclusion as an oplog frame. After a laptop restore, the gateway pulled missing frames and rebuilt its local store.

### 2.2 Vector-clock watermarks

**Statement:** Track causality with per-actor sequence counters instead of wall-clock timestamps.

**Why it helps:** Concurrent writes can be ordered or flagged; last-write-wins semantics do not silently erase agent intent.

**When to use:** When multiple edges write concurrently and you need deterministic conflict ordering.

**Example.** Two edges updated the same peer card simultaneously. The gateway compared vector clocks and preserved the loser in a conflict ledger.

### 2.3 Tombstone soft deletes

**Statement:** Represent deletion as an oplog frame plus a soft-delete marker, retained for a bounded window.

**Why it helps:** Deleted facts stay deleted after sync; tombstones can be garbage-collected once the retention window passes.

**When to use:** When deletions must propagate to offline peers without reappearing.

**Example.** An agent deleted a stale conclusion on one edge. The tombstone frame reached a second edge that had been offline for a day.

### 2.4 Conflict ledger

**Statement:** Persist losing versions alongside the winner so agents or operators can review and merge.

**Why it helps:** No silent data loss; conflicts become observable and actionable.

**When to use:** When vector clocks diverge without a clear ancestor and automatic resolution is risky.

**Example.** A concurrent peer-card update produced divergent facts. The Verifier exposed both versions through a conflict review endpoint.

### 2.5 HTTPS push/pull sync agent

**Statement:** A local daemon that pushes new oplog frames to the hub and pulls missing frames on reconnect.

**Why it helps:** Agents keep working offline; the hub reconciles writes when connectivity returns.

**When to use:** When edges are intermittently online and need asynchronous, durable sync.

**Example.** A fleet edge wrote facts while disconnected. The sync agent uploaded them after the transport came back, with no duplicate frames due to idempotent keys.

### 2.6 Idempotent operation keys

**Statement:** Make each oplog frame uniquely addressable so retries are safe.

**Why it helps:** Retries after timeouts do not create duplicate facts or corrupt vector clocks.

**When to use:** When push/pull happens over unreliable networks.

**Example.** The Archivist insisted that every frame key include edge identity and sequence. A retried push produced exactly one row on the hub.

## 3. Anti-patterns

### 3.1 Blocking-embedding path

**Statement:** Blocking the memory write until the embedding model returns a vector.

**Consequence:** Agents experience multi-second write latency; many stop calling the memory tool and keep facts in prompt context or scratchpads.

**Example.** In the previous backend, a remote embedding server had to respond before a write could return. Agents began keeping facts in local scratchpads instead.

### 3.2 Heavy default stack

**Statement:** Requiring a relational fleet store, vector fleet store, remote embedding server, in-memory store, and container orchestrator for local operation.

**Consequence:** A single developer cannot run the stack; operational surface becomes enormous relative to the value of storing one fact.

**Example.** The old deployment needed five services and an orchestrator before the first fact could be stored. The local agent never adopted it.

### 3.3 Anonymous-agent default

**Statement:** Falling back to shared default agent and user identities when caller IDs are omitted.

**Consequence:** Shared memories become misattributed; concurrent agents overwrite each other's conclusions; provenance is lost.

**Example.** Two agents both wrote to the same default identity. The Verifier later could not tell which observation belonged to which agent.

### 3.4 Low-level primitives as the only surface

**Statement:** Forcing agents to understand identities, sessions, conclusions, and watermarks for everyday remember/recall tasks.

**Consequence:** Agents write brittle wrapper logic or skip the backend entirely.

**Example.** A skill needed three tool calls and manual identity and session bookkeeping to store a single insight. Most agents kept the insight in their prompt context instead.

### 3.5 CRDT overkill

**Statement:** Applying full commutative/replicated-data-type semantics to data that is not naturally commutative.

**Consequence:** Complex algebra, subtle semantic divergence, and high engineering cost for mostly append-only memory.

**Example.** The Researcher prototyped a CRDT for session transcripts. Message ordering diverged across edges despite structural convergence.

### 3.6 Primary-replica for append-mostly data

**Statement:** Using a leader-follower database topology as the default sync mechanism.

**Consequence:** Edges cannot write offline; recovery requires reasoning about leader election and split-brain.

**Example.** An earlier design put a fleet relational store leader in the hub. An offline edge could not create facts and had to buffer them externally.

### 3.7 Notification-as-source-of-truth

**Statement:** Using an in-memory publish/subscribe channel as the sync transport.

**Consequence:** Offline peers miss updates; the only safe path becomes a full re-pull on reconnect.

**Example.** A previous event design used the in-memory store's pub/sub for fleet events. A rebooted gateway missed updates that were never persisted.

### 3.8 Embedded-replica trap

**Statement:** Treating an embedded database's frame-level replication as the sync wire protocol.

**Consequence:** The application must quiesce local reads during sync windows; operators must understand two sync abstractions.

**Example.** One backend candidate warned that opening the local database during replication could corrupt it. The gateway could not pause all reads, so the embedded replication path was rejected as default.

### 3.9 Last-write-wins as default conflict policy

**Statement:** Resolving conflicts by wall-clock timestamp alone.

**Consequence:** Concurrent updates to conclusions or peer cards can be silently discarded.

**Example.** An earlier sync strategy chose the later wall-clock timestamp. A peer-card update made on a clock-skewed edge overwrote a more accurate fact.

### 3.10 Single-transport assumption

**Statement:** Hard-coding a single network overlay as the only supported transport.

**Consequence:** Operators who prefer direct IP, WireGuard, SSH tunnels, or mTLS over public networks are excluded.

**Example.** Early wording implied the overlay was the only exposure path. The final architecture treats the overlay as optional convenience and supports any secure transport.

### 3.11 Convenience-first security default

**Statement.** Allowing anonymous or weak-default authorisation when the gateway listens on a non-local address.

**Consequence.** A lost device or compromised local network exposes memory plaintext.

**Example.** The old stack had a documented weak default signing secret. The new design requires operator-set secrets before fleet mode and refuses anonymous remote mode.

## 4. Pattern-to-paper index

| Pattern / anti-pattern | Papers |
|---|---|
| full-text-first retrieval | 06-sync-taxonomy, 07-local-first-runtime, 08-explicit-identity |
| Async embedding queue | 07-local-first-runtime, 08-explicit-identity |
| Single local runtime package | 07-local-first-runtime |
| Append-only operation log | 06-sync-taxonomy, 09-federated-failure-modes |
| Vector-clock watermarks | 06-sync-taxonomy, 09-federated-failure-modes |
| Tombstone soft deletes | 09-federated-failure-modes |
| Conflict ledger | 06-sync-taxonomy, 09-federated-failure-modes |
| HTTPS push/pull sync agent | 06-sync-taxonomy, 07-local-first-runtime |
| Idempotent operation keys | 06-sync-taxonomy |
| Blocking-embedding path | 07-local-first-runtime, 08-explicit-identity, 09-federated-failure-modes |
| Heavy default stack | 07-local-first-runtime |
| Anonymous-agent default | 08-explicit-identity, 09-federated-failure-modes |
| Notification-as-source-of-truth | 09-federated-failure-modes |
| Embedded-replica trap | 06-sync-taxonomy, 09-federated-failure-modes |
| Last-write-wins default | 06-sync-taxonomy |
| Single-transport assumption | 06-sync-taxonomy, 07-local-first-runtime |
| Convenience-first security default | 09-federated-failure-modes |

## 5. Glossary

A short lexicon for recurring fleet-memory terms used in this paper set.

| Term | Meaning |
|---|---|
| **Operation log / oplog / event log** | An append-only sequence of immutable frames describing memory writes, each carrying payload, actor, and causal metadata. |
| **Frame** | One immutable entry in the operation log. |
| **Watermark** | A per-actor sequence counter used to establish causal ordering between frames. |
| **Vector-clock watermark** | A set of per-actor watermarks that together capture causality across multiple edges. |
| **Peer card** | A structured profile summarising what the fleet knows about a peer or agent. |
| **Conflict ledger** | A store of divergent versions produced by concurrent writes, kept for review rather than silently merged. |
| **Tombstone** | A soft-delete frame that prevents a deleted fact from resurrecting on offline peers. |
| **Sync agent** | A local daemon that pushes and pulls operation-log frames between an edge and a fleet hub. |
| **Hub identity pinning** | Binding an edge to a specific, authenticated hub identity so watermarks remain meaningful. |
| **full-text search** | Full-text search; the default retrieval mode in the local-first gateway. |
| **Fleet mode** | A gateway configuration in which the local store synchronises with a shared fleet hub. |
| **Local-first** | A design in which the edge stores, reads, and writes durable memory before any network dependency is resolved. |
