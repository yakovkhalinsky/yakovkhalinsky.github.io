# Memory-Centric Collaboration: A Position Paper

**Abstract.** We argue that scaling long-running multi-agent fleets depends more on shared persistent state than on orchestration logic. Orchestration can route tasks; only memory gives a fleet continuity, identity, and the ability to recover. We outline five principles for memory-centric collaboration and show how they change the design of both agent tools and fleet architecture.

## 1. The argument

Most discussions of multi-agent systems focus on routing: how to decide which agent does what. Routing is important, but it is not sufficient. A fleet that routes perfectly among agents with no shared memory is still a collection of isolated assistants. Each conversation starts from near zero. Each hand-off requires re-explaining context. Each restart forgets what the fleet learned.

The dominant scaling problem in our experience has been memory. Not model capability, not tool diversity, not prompt engineering. The ability of a fleet to remember, attribute, and share what it learned.

## 2. Five principles

### 2.1 Externalise session state

An agent's working memory should live outside the agent process. Prompt context, local files, and process-local variables are convenient but fragile. Durable session state lets an agent restart, upgrade, or move between hosts without losing the thread.

### 2.2 Explicit identity

Every memory operation must name who is remembering and on whose behalf. Defaults that silently partition data by session, role, or agent create the illusion of a shared memory while actually maintaining separate silos. Explicit identity is the prerequisite for correct recall.

### 2.3 Immutable goal artefacts

Goals, plans, and decisions should be stored as versioned artefacts, not as transient messages. A message can be summarised away; an artefact remains and can be referenced, audited, and reverted. The fleet's long-term memory is its artefact graph, not its chat history.

### 2.4 Shared event log

Coordination events should be written to a shared, append-only log that every role can read. The log is the source of truth for who did what and why. It replaces ad-hoc status messages and makes hand-offs auditable.

### 2.5 Local-first, shared-when-useful

A memory surface should work locally by default and federate when useful. Local mode gives developers speed and debuggability. Fleet mode adds durability and cross-agent visibility. The same agent surface should support both without code changes.

## 3. Implications for tool design

Memory tools should be high-level, not raw database operations. An agent should be able to remember a fact, recall related facts, and search across conclusions and sessions without learning a schema. The surface should handle identity, attribution, and search ranking so the agent can focus on reasoning.

Tools should also distinguish local draft memory from shared fact. Promotion from one to the other should be explicit. Otherwise, every half-formed thought becomes noise in the shared store.

## 4. Implications for fleet architecture

A memory-centric fleet has a different centre of gravity. The shared memory layer becomes the primary correctness and recovery surface. The orchestrator routes tasks, but the memory layer records ownership, state, and history. If the orchestrator fails, the fleet can resume from memory. If memory fails, the orchestrator has nothing to route.

This does not mean the orchestrator is unimportant. It means the orchestrator's design should follow from the memory contract, not the other way around.

## 5. Limits and counterarguments

Memory-centric collaboration is not a universal recipe. Some tasks are stateless and do not need durable memory. Some fleets are ephemeral and would be burdened by persistent state. Some domains have strict privacy constraints that make shared memory hard.

We also acknowledge that good memory can mask bad design. A fleet that remembers every mistake but never fixes the underlying cause will archive its failures faster than it learns from them. Memory must be paired with feedback loops: verification, rollback, and periodic review.

## 6. Conclusion

The argument of this paper is simple: long-running agent fleets scale by sharing state, not by adding agents. The role taxonomy, failure modes, and evaluation protocol in this set all rest on the assumption that durable, attributed, searchable memory is the foundation of collaboration. Builders who start with memory design will find that orchestration becomes easier; those who start with orchestration will find that memory failures keep undoing their routing decisions.

## References

- [Agentic Strategies and Patterns Catalog](../artifacts/agentic-patterns.md)
- [Incident Archetypes](../artifacts/incident-archetypes.md)
- [Role Taxonomy](../artifacts/role-taxonomy.md)
- [Bibliography](../artifacts/bibliography.md)
