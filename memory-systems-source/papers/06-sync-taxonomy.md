# A Taxonomy of Sync Strategies for Fleet Memory

**Abstract.** We compare six sync strategies for shared memory in long-running multi-agent fleets: append-only operation log (oplog) sync, embedded database replication, consensus / Raft, conflict-free replicated data types (CRDTs), primary–replica replication, and last-write-wins. We evaluate each strategy on five dimensions that matter for fleet memory: offline tolerance, conflict model, operator burden, recovery simplicity, and the identity of the sync abstraction. Append-only oplog sync with vector-clock watermarks closely matches the append-mostly, identity-sensitive, intermittently connected workloads typical of agent fleets. We identify the threats and open questions that remain.

## 1. Introduction

Long-running agent fleets need shared memory that survives restarts, hand-offs, and intermittent connectivity. The dominant design question is not *whether* to synchronise, but *which* sync contract to adopt. The wrong contract creates silent failures: memories disappear, identities collide, concurrent updates are silently discarded, or operators must reason about two different sync abstractions at once.

This paper surveys six strategies that could plausibly be used to keep fleet memory consistent. We do not compare product implementations; we compare design contracts. Our goal is to give builders a vocabulary for the decision and to explain why an append-only operation log (oplog) with vector-clock watermarks emerged as the natural default for the fleet-memory workload.

## 2. Why sync is the hard part of fleet memory

Fleet memory has three properties that make sync difficult.

First, **edges are intermittently online**. Agents run on laptops, edge devices, and transient cloud instances. A sync strategy that requires a live leader or a continuous connection will stall the agent whenever the network flakes.

Second, **the workload is append-mostly but not append-only**. Most writes are new facts, messages, and conclusions. But agents also update peer cards, correct stale conclusions, and delete draft notes. The conflict model must handle the rare update carefully, not optimise for it at the expense of the common append case.

Third, **identity and provenance matter as much as the data value**. A memory is not just a key–value pair; it records who observed it, on whose behalf, in which session. A sync strategy that resolves conflicts without preserving the losing version makes auditing and correction impossible.

These three properties mean that fleet memory sync is not a pure database-replication problem. It is a distributed systems problem with strong operator-recovery and provenance requirements.

## 3. Dimensions of comparison

We compare strategies along five dimensions.

| Dimension | Question |
|---|---|
| **Offline tolerance** | Can an edge write and read locally while disconnected? |
| **Conflict model** | How are concurrent or divergent writes resolved, and is the resolution visible? |
| **Operator burden** | How many concepts must an operator understand to debug a sync problem? |
| **Recovery simplicity** | How does an operator recover a stale or corrupted edge? |
| **Identity of sync abstraction** | Is the sync contract the application-level operation log, the storage engine, a consensus protocol, or something else? |

These dimensions are not orthogonal. A strategy with a simple abstraction, such as last-write-wins, usually has a weak conflict model. A strategy with a strong conflict model, such as a full CRDT, usually raises operator burden. The appropriate choice depends on which dimensions the workload stresses most.

## 4. Strategies compared

### 4.1 Append-only operation log (oplog) sync

In this strategy, every write is recorded as an immutable frame containing the payload, the actor identity, a vector-clock watermark, and optional causal metadata. Local writes append a frame to the local store and apply the payload to the local working view. A sync agent periodically pushes new frames to the hub and pulls frames the hub has that the local replica lacks.

**Offline tolerance.** Excellent. Edges write to the local log immediately and read from the local materialised view. Sync is asynchronous; an edge can be offline for hours or days and still function.

**Conflict model.** Concurrent writes are preserved. Vector-clock comparison determines causal order; writes that are not causally related are both kept or both exposed in a conflict ledger. Intentional supersession is declared explicitly by a causal pointer rather than inferred from a timestamp.

**Operator burden.** Low to moderate. The operator thinks in frames, watermarks, and replay. There is one sync abstraction: the oplog.

**Recovery simplicity.** High. A stale edge pulls missing frames and replays them. A corrupted edge can be reseeded from the hub or restored from a snapshot. There is no leader election or hidden replication state to debug.

**Identity of sync abstraction.** The sync abstraction is the application-level operation log. The storage engine is just where the log is kept.

**Anonymised example.** A Runtime role keeps a gateway running on a laptop. During a transport outage the local agent writes fifty conclusions to the local oplog. When connectivity returns, the sync agent pushes those frames to the hub, and a second edge later pulls them. No frame is lost or duplicated because each frame has a deterministic key.

### 4.2 Embedded database replication

Here an embedded database engine, such as a file-backed relational replica, replicates its storage pages or transaction-log frames directly from a primary. The application reads and writes through the embedded engine, and replication is handled below the application layer.

**Offline tolerance.** Moderate. Reads and writes may be possible locally, but the exact semantics depend on the engine and its replication mode.

**Conflict model.** Usually determined by the database engine. Last-write-wins or primary-wins is common; concurrent updates may be lost or require engine-specific handling.

**Operator burden.** Moderate to high. The operator must understand both the storage engine’s replication mechanics and the application’s data model. One vendor’s embedded-replica documentation warns that opening the local database while the replica is syncing can corrupt data, which forces the application to quiesce reads during sync windows.

**Recovery simplicity.** Moderate. Reseeding from the primary is often supported, but operators must reason about replica lag, sync windows, and possible split-brain if the primary changes.

**Identity of sync abstraction.** The sync abstraction is the storage engine itself. Application logic and storage replication are entangled.

**Anonymised example.** A Researcher prototyped a fleet gateway using an embedded replica engine. The prototype worked in demos, but during a long sync window the gateway could not serve local reads without risking corruption. The team rejected the approach as the default because it forced two sync abstractions onto operators.

### 4.3 Consensus / Raft

A consensus protocol such as Raft keeps a replicated log across a cluster of nodes. Writes are committed when a quorum agrees. This gives strong consistency and fault tolerance.

**Offline tolerance.** Poor for edge nodes. A disconnected edge cannot participate in quorum and therefore cannot commit writes. It must either block or fall back to a different mode.

**Conflict model.** Strong consistency at the cost of availability. Conflicts are avoided by serialising through the leader; concurrent writes at the edge are not naturally supported.

**Operator burden.** High. Operators must understand leader election, quorum size, log compaction, and membership changes.

**Recovery simplicity.** Moderate. A new or failed node rejoins, receives a snapshot and log entries, and catches up. But if an edge has been offline too long, it may need a full snapshot transfer, and operators must manage cluster membership.

**Identity of sync abstraction.** The sync abstraction is the consensus protocol and its replicated log. The application must map its operations onto consensus rounds.

**Anonymised example.** A design discussion considered using a consensus group for the fleet hub. The Runtime role pointed out that an edge on a laptop cannot form a quorum with the hub during a network partition. The approach was kept as a possible hub-internal mechanism but rejected for edge-to-hub sync.

### 4.4 Conflict-free replicated data types (CRDTs)

CRDTs are data structures designed so that concurrent updates can be merged without coordination. They are attractive for always-available, peer-to-peer systems.

**Offline tolerance.** Excellent. Edges can write locally and merge later.

**Conflict model.** Convergence is guaranteed, but the *meaning* of the merged state may not match agent intent. Messages, ordered lists, and intentional replacements do not map cleanly to commutative operations.

**Operator burden.** High for non-commutative data. Designing correct CRDTs for session transcripts, peer cards, and conclusion supersession requires careful algebra and often custom data types.

**Recovery simplicity.** Moderate. Converging state is automatic, but diagnosing *why* two edges converged to a particular merged state can be difficult because the algebra is hidden from operators.

**Identity of sync abstraction.** The sync abstraction is the data type itself. Correctness depends on every writer using the same CRDT semantics.

**Anonymised example.** The Researcher prototyped a CRDT for session messages. Structural convergence held, but message ordering diverged across edges: each edge saw a valid but different transcript. The team concluded that full CRDTs were overkill for mostly append-only memory data.

### 4.5 Primary–replica replication

In primary–replica replication, one node is the write leader and other nodes are followers. Reads may be served from replicas, but writes go to the primary.

**Offline tolerance.** Poor for writes. An offline edge cannot write to the primary and therefore cannot create new fleet memory unless it buffers writes externally.

**Conflict model.** Simple but lossy. The primary’s value wins; concurrent writes at the edge may be overwritten or rejected.

**Operator burden.** Moderate. Operators must track the primary, replica lag, and failover procedures.

**Recovery simplicity.** Moderate. A failed primary requires promotion; a stale replica re-syncs from the primary. Split-brain is a constant risk.

**Identity of sync abstraction.** The sync abstraction is the database topology: primary, replicas, and replication stream.

**Anonymised example.** An earlier fleet design placed a relational store leader in the hub and made edges read-only replicas. An offline edge that needed to record a finding had to buffer it in a local scratchpad and copy it later, which introduced duplicate and lost entries.

### 4.6 Last-write-wins

Last-write-wins resolves conflicts by keeping the write with the latest timestamp. It is the simplest distributed sync strategy.

**Offline tolerance.** Excellent. Edges can write locally and resolve conflicts when they reconnect.

**Conflict model.** Dangerous for agent memory. Clock skew, concurrent updates, and intentional supersession are all invisible. A peer-card update made on a skewed clock can overwrite a more accurate fact.

**Operator burden.** Very low. The concept is easy to explain.

**Recovery simplicity.** High, but for the wrong reason: recovery is simple because losing data is simple.

**Identity of sync abstraction.** There is no separate abstraction; sync is implied by timestamp comparison.

**Anonymised example.** An early sync strategy chose the later wall-clock timestamp whenever two edges updated the same peer card. A clock-skewed edge overwrote a carefully verified correction, and the Verifier had no record of the discarded version.

## 5. Decision guide

Table 1 summarises the comparison.

**Table 1. Comparison of sync strategies for fleet memory.**

| Strategy | Offline tolerance | Conflict model | Operator burden | Recovery simplicity | Identity of abstraction |
|---|---|---|---|---|---|
| Append-only oplog | Excellent: local writes and reads work offline | Causal / vector-clock, explicit supersession, conflict ledger | Low–moderate | High | Application operation log |
| Embedded DB replication | Moderate: depends on engine and mode | Engine-defined; often last-write-wins | Moderate–high | Moderate | Storage engine replication |
| Consensus / Raft | Poor at edge; strong at hub | Strong consistency via quorum | High | Moderate | Consensus protocol and replicated log |
| CRDTs | Excellent | Guaranteed convergence, semantic risk | High for non-commutative data | Moderate | Data-type algebra |
| Primary–replica | Poor for writes | Primary wins, lossy | Moderate | Moderate | Database topology |
| Last-write-wins | Excellent | Timestamp-based, dangerous | Very low | High (because it discards) | None (timestamp) |

The decision flow can be stated in prose:

- If the data is **append-mostly and identity-sensitive**, prefer the **append-only oplog**.
- If the deployment is **always online and writes are rare**, **embedded replication** or **primary–replica** may suffice.
- If the fleet hub needs **strong consistency across a small cluster**, **consensus** is appropriate for the hub internals, but not for edge-to-hub sync.
- If the data is **naturally commutative** and semantic drift is acceptable, consider a **CRDT**.
- Avoid **last-write-wins** as the default for any data where losing a write would surprise an operator or agent.

## 6. Which strategy fits the fleet-memory workload

For the append-mostly, identity-sensitive, intermittently connected workload typical of long-running agent fleets, the append-only oplog with vector-clock watermarks is the natural default. The reasons are structural, not merely preferential.

First, the workload is append-mostly. New conclusions, messages, and facts dominate; they map cleanly to immutable frames.

Second, edges must keep working offline. Agents cannot pause while they wait for a quorum, a primary, or a sync window.

Third, identity and provenance are first-class. Every frame carries the actor and the observed subject. The conflict ledger preserves losing versions, so the Verifier can audit what happened.

Fourth, recovery must be operator-simple. A stale or corrupted edge runtime is recovered by pulling the oplog or restoring a snapshot. There is no hidden replication state to reason about.

Fifth, the sync abstraction must be singular. Operators debug frames and watermarks, not “the storage engine’s frame-level replication plus the application’s oplog.” Decoupling sync from storage keeps the contract clear.

We do not reject the other strategies absolutely. Consensus may still be useful inside the fleet hub for small strongly consistent metadata. Embedded replication remains a possible future optimisation. CRDTs may be appropriate for specific commutative data structures. But for the core fleet-memory contract, the oplog is the defensible default.

## 7. Threats and open questions

This default has its own weaknesses.

**Oplog growth.** An append-only log grows forever unless garbage collection prunes old frames and tombstones. A fixed retention window bounds offline capability: an edge offline longer than the window must reseed. The trade-off between storage cost and offline duration must be explicit.

**Single-writer concurrency at the edge.** A file-backed local database cannot accept concurrent writes from many agents without serialisation. The edge runtime must funnel writes through a single writer or shard by workspace.

**Vector-clock overhead.** Tracking causality with vector clocks adds metadata and comparison cost. At very large scale, the overhead may require compression or hybrid clocks.

**Conflict visibility is only half the answer.** Preserving conflicts in a ledger is useful only if agents or operators actually review them. Automatic silent resolution is avoided, but manual resolution introduces its own latency and burden.

**Transport assumptions.** The oplog can run over a secure transport, but the protocol must still authenticate the hub identity, handle network partitions, and bound retry backlog. A secure transport is not the same as a correct sync protocol.

**Empirical validation is limited.** The comparison in this paper is analytical, drawn from one design cycle and one operational fleet. Other builders should replicate the evaluation on their own workloads before treating the ranking as more than a field-derived hypothesis.

## 8. Conclusion

Sync is the hard part of fleet memory because agent memory is append-mostly, intermittently connected, and identity-sensitive. We compared six sync strategies on offline tolerance, conflict model, operator burden, recovery simplicity, and the identity of the sync abstraction. The append-only oplog with vector-clock watermarks closely matches the fleet-memory workload: it lets edges work offline, preserves provenance and conflicts, simplifies recovery, and gives operators a single sync abstraction to debug. Other strategies have valid niches, but none should be the default for the core shared-memory contract in a long-running agent fleet.

## References

See [fleet-memory bibliography](../artifacts/fleet-memory-bibliography.md).
