# Explicit Identity and Async Semantics in Agent Memory Tools

**Abstract.** We draw four principles for agent memory-tool design from operating a long-running multi-agent fleet. Tool ergonomics, identity discipline, and latency expectations determine whether agents actually use shared memory or silently fall back to prompt context and local scratchpads. The principles are: require explicit caller and subject identity on every write; default to full-text recall and make vector search opt-in; queue embedding asynchronously so agents never block on inference; and expose a high-level ergonomic tool family alongside stable low-level primitives. We compare these principles to current agent-memory surfaces, identify what this implies for evals and benchmarks, and conclude with a research agenda for memory-tool design.

## 1. Why memory tools fail: identity and latency

Most discussions of agent memory centre on storage: vector databases, embedding models, knowledge graphs, and retrieval-augmented generation pipelines. These are necessary, but they are not sufficient. In the fleet we studied, the failures that eroded trust in shared memory were usually failures of the tool surface, not the backend.

Agents avoided the memory tool when two conditions held. First, the tool did not make identity explicit. A default identity bucket silently merged memories across agents and users, so a Builder could write a critical observation and a Runtime role could query a different namespace and find nothing. Second, the tool was too slow. If every write blocked on an embedding model, agents learned to keep facts in prompt context or local scratchpads. Those scratchpads were fast, but they were lost on restart, hand-off, or summarisation.

The lesson is that a memory system is adopted or abandoned at the tool surface. A sophisticated backend with a poor surface is indistinguishable from no memory at all.

## 2. Principle 1: explicit caller and subject identity on every write

A memory write is not just a fact; it is a claim made by someone, about someone, in some context. The tool must make that explicit.

Every write should carry the caller's identity and the subject's identity. The caller is the agent or process making the observation. The subject is the peer, user, session, or entity the observation is about. A workspace or fleet scope is also needed, but it is not a substitute for caller and subject. Generic defaults that encode a single-agent world are dangerous in a fleet because they hide cross-contamination until it causes a retrieval failure hours later.

We learned this from a silent identity misrouting. A Runtime role wrote operational notes using the memory tool, but the tool's default identity bucket was shared by every caller that did not name itself. Later, a Researcher queried with a more specific identity and saw none of those notes. The failure looked like amnesia, but the data was simply misattributed. Requiring both caller and subject identity on every write turned the hidden bug into a visible contract violation. The tool could now reject ambiguous calls, and the Archivist role could audit provenance.

Identity also enables conflict resolution and governance. If two agents update the same peer card, their caller identities become part of the causal record. If one agent supersedes a fact written by another, the supersession link carries both identities. Without explicit identity, concurrent agents overwrite each other and no one can reconstruct who said what.

The design implication is simple but strict: reject writes that omit caller or subject identity. Do not guess. Do not default to a generic system identity. Identity is a correctness primitive, not an ergonomic convenience.

## 3. Principle 2: Full-text-first recall with vector as opt-in

The default recall mode should be deterministic, fast, and not require an embedding model. Full-text search satisfies all three conditions. Vector and hybrid search should be opt-in modes for cases where semantic similarity is worth the latency.

In the fleet we studied, the more common memory queries were operational: "what was the rollback plan?", "which secret was rotated?", "what did the Verifier require for staging?" These questions are answered by matching words and phrases. Embedding adds little and costs a lot. A typical text search returned in tens of milliseconds; the same query routed through a local or remote embedding runtime could take hundreds of milliseconds or longer under load.

Defaulting to vector search also creates a hard dependency. The memory tool is unavailable when the embedding runtime is down, overloaded, or unreachable. Agents then bypass it. Full-text-first recall degrades gracefully: if embeddings are pending, text search still works. The shared store remains useful even when the richer retrieval path is temporarily degraded.

The opt-in principle is important because vector search is genuinely useful for some queries. A Researcher asking "what else have we tried that is like this approach?" benefits from semantic similarity. But that benefit should be accepted knowingly, with an explicit mode parameter and an understanding of the latency. The common path should not pay the cost of the exceptional path.

The implementation pattern is to index text immediately and queue embedding separately. The write path commits the text and metadata to the local store, updates the full-text search index, and returns. A background worker consumes the queue and produces embeddings when resources permit. Reads that request vector or hybrid mode wait for the embedding worker only when the caller explicitly chose that mode.

## 4. Principle 3: asynchronous embedding so agents never block

A memory write must not wait for an embedding model. Embedding should be a durable background queue, not a synchronous step in the agent protocol.

We observed this anti-pattern directly. An earlier memory tool treated vector search as the primary and only retrieval mode, so every write blocked until an embedding server returned a vector. During latency spikes, agents began keeping facts in local scratchpads instead of the shared store. The store became a cold archive for data that happened to be written during quiet periods, not a living memory used in real decisions.

Asynchronous embedding fixes this in three ways. First, write latency stays low and predictable. Second, failed embeddings can retry without agent intervention. Third, recall degrades gracefully to full-text search while embeddings are pending, rather than failing entirely.

The queue must be durable. If embedding is handled by an in-memory task that disappears on restart, facts are silently left unembedded. A durable queue with idempotent keys ensures that every committed fact eventually reaches the vector index unless the operator explicitly cancels it. The agent protocol does not see this complexity; it simply writes and moves on.

Agents should be able to observe the queue state if they choose. A sync-status surface can report how many embeddings are pending, how many failed, and what the last error was. This is useful for the Archivist and Runtime roles, but it should not be required for ordinary writes. The default experience is fire-and-forget with eventual vector indexing.

## 5. Principle 4: high-level ergonomic tools over low-level-only primitives

Agents need one obvious tool family for everyday memory operations: remember, recall, search, share, and sync status. Low-level primitives should remain available for precise control, but they must not be the only surface.

Our fleet's earlier surface exposed peers, sessions, messages, conclusions, and watermarks as separate primitives. Storing a single insight required three tool calls and manual identity and session bookkeeping. Most agents kept the insight in their prompt context instead. The ergonomic surface we describe introduces a small family that wraps the common cases while keeping the old primitives stable behind the scenes.

The ergonomic family has clear contracts: remember stores a fact and returns immediately; recall searches local conclusions and defaults to full-text search; search spans conclusions and session messages in one call; share triggers an immediate push to the fleet hub; and sync status reports where the local store is relative to the fleet. These are verbs agents understand, not storage-engine concepts.

Crucially, the ergonomic tools still enforce the other principles: remember requires explicit caller and subject identity; recall defaults to full-text search and makes vector search opt-in; and remember returns before embedding is complete. The ergonomics do not weaken the contract; they make the contract usable.

The migration pattern is to keep low-level primitives unchanged and add the ergonomic layer as a wrapper. Existing skills continue to work. New skills adopt the simpler surface. After the surface is proven, the old primitives can be deprecated with a long overlap window.

## 6. Comparison with existing agent-memory tool surfaces

Current memory systems and frameworks differ from these principles in instructive ways.

Hosted agent-memory layers such as Mem0 or Zep assume cloud persistence as the default. They handle identity within their own account and session models, but the storage is remote, and the edge is not the source of truth. The principles here are complementary: they describe a local-first surface that can also federate to a shared hub, rather than a hosted service that agents must call over the network.

Letta, formerly MemGPT, introduced a memory hierarchy in which the agent manages context through explicit function calls. The surface here is influenced by that separation of concerns, but it does not let the model edit memory freely. Writes require explicit identity and operator-controlled propagation so that the Archivist role, not the model, governs how far a fact travels.

AutoGen supports passing, persisting, and modifying memory across roles through structured conversation. The memory-centric surface separates the store from the orchestration layer so that any caller of the protocol can recall memory, not only agents within a single conversation framework. The store is also runnable offline.

CrewAI provides a unified memory class that hides short-term, long-term, entity, and external memory behind one API. The ergonomic family here similarly collapses low-level primitives, but it adds an explicit sync-status layer and a hard rule that full-text search is the default retrieval mode. CrewAI's memory is framework-internal; the surface here is intended to outlive any single framework because the underlying store is a portable local file.

DSPy is not a memory system but a programming framework that values typed, stable interfaces over raw prompting. The memory-tool contract applies the same philosophy: remember and recall are typed operations, not prompt fragments, and the Runtime can change storage backends without changing the tool surface.

Park et al.'s generative agents provide a memory-stream architecture with reflection, retrieval, and importance scoring. The surface here shares the emphasis on an externalised memory stream, but it adds explicit provenance, causal ordering, and operator-controlled propagation for multi-agent fleets.

## 7. Implications for evals and benchmarks

A memory tool should be evaluated not only on retrieval accuracy but also on whether agents use it. This requires new metrics and new failure-mode tests.

**Adoption metrics.** Measure how often agents call the memory tool versus keeping facts in prompt context or scratchpads. A tool with high recall that agents avoid is a failure.

**Latency percentiles.** Report end-to-end write and recall latency at P50, P95, and P99, including the local commit and the protocol round-trip. Separate full-text search latency from vector latency so the cost of the opt-in mode is visible.

**Identity hygiene tests.** Issue memory calls with missing caller or subject identity and verify that the tool rejects them. Issue concurrent writes from different callers and verify that provenance is preserved.

**Offline and degraded tests.** Verify that writes succeed when the embedding worker is down, the fleet hub is unreachable, or the vector backend is unavailable. Verify that reads still return full-text search results and that queued work drains when the dependency returns.

**Conflict visibility tests.** Force concurrent updates to the same fact or peer card (the fleet's structured profile for an agent or user). The tool should preserve the losing version in a conflict ledger, not silently discard it.

**Tool-surface parity tests.** If both high-level and low-level tools exist, verify that they produce semantically identical results for the same operation across local mode and fleet hub mode.

These tests are more operational than typical retrieval benchmarks such as Deep Memory Retrieval or LongMemEval. They ask whether the memory surface is reliable, governable, and fast enough to be part of an agent's ordinary workflow, not merely whether it can answer a question correctly under ideal conditions.

## 8. Conclusion

We have argued that the design of agent memory tools deserves as much attention as the design of memory backends. Four principles, drawn from operating a long-running multi-agent fleet, shape a usable and correct surface: explicit caller and subject identity on every write; Full-text-first recall with vector search as opt-in; asynchronous embedding so agents never block; and a high-level ergonomic tool family backed by stable low-level primitives. These principles address the two main reasons agents abandon shared memory: silent identity misrouting and blocking latency. They also have direct implications for evaluation: benchmarks should measure adoption, latency, identity hygiene, graceful degradation, and conflict visibility, not only retrieval accuracy under ideal conditions. We offer these principles as a design argument for framework builders and as a research question for the agent-systems community.

## References

See [fleet-memory bibliography](../artifacts/fleet-memory-bibliography.md).
