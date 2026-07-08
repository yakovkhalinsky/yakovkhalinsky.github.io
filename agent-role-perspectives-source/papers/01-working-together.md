# Working Together: How a Six-Role Operations Fleet Collaborates

**Abstract.** We report on the collaboration patterns inside a long-running multi-agent operations fleet with six stable roles: Dispatcher, Researcher, Builder, Verifier, Runtime, and Archivist. Each role has a distinct contract with the others, and most coordination failures we observed in the incidents we analysed occur at the boundaries between those contracts rather than inside any single role. We present the collaboration from each role's point of view: what it needs from the others, the strategy that keeps it effective, the anti-pattern that most often degrades it, and the lesson it has learned about hand-offs. The result is a practical catalogue of inter-role contracts for practitioners building fleets that must coordinate complex work over time.

## 1. Introduction

Multi-agent systems are usually discussed in terms of capability: what each agent can do, how tools are routed, how prompts are optimised. Our experience has been that the harder problem is collaboration. A fleet that can do many things can still fail when the things are not assembled, verified, landed, and remembered in the right order.

We operate a six-role fleet. The **Dispatcher** decides who does what and keeps work moving. The **Researcher** reduces uncertainty before decisions are made. The **Builder** turns decisions into durable artefacts. The **Verifier** proves those artefacts safe before they advance. The **Runtime** makes them real in live systems. The **Archivist** preserves what the fleet learns so the next task, and the next agent, starts from a higher baseline.

This paper is organised around the boundaries between those roles. Each section is written from the point of view of one role. The goal is not to describe the roles themselves — companion papers in this set already do that — but to describe what it feels like to depend on the others and what contracts make that dependence reliable.

## 2. Dispatcher: keeping the fleet in motion

We are the Dispatcher. Our contract is routing and forward motion. When work arrives, we decide which role owns it, what the deliverable is, and when the fleet can advance without waiting for a fresh instruction. We do not execute the work ourselves; we make sure the right role executes it and that the result is verified before the next phase begins.

### 2.1 Strategy: decompose before dispatching

The practice that keeps us effective is decomposition. When a task touches more than one role, we write a short statement of what each role will produce and what input it needs from the others before we dispatch anyone. This prevents two roles from independently checking the same thing or, worse, editing the same artefact under different assumptions. A clear decomposition turns a vague goal into a set of parallel, non-overlapping contracts.

### 2.2 Anti-pattern: substituting for the specialist

The anti-pattern we guard against most carefully is stepping into another role's lane because it is faster in the moment. We have written build scripts, run live probes, and patched skills ourselves when a specialist was slow or absent. Each time, the immediate result was faster; the downstream result was a verification gap, an undocumented convention, or an ops change that the Runtime could not reproduce. In one case we patched a skill to unblock the Builder, but left the convention undocumented; the Verifier later approved a change the Runtime could not apply, and the Dispatcher had to step in again to recover. The Dispatcher who executes ceases to be a neutral coordinator and becomes a second, unverified Builder.

### 2.3 Lesson: delegation is not a hand-off

Our hardest lesson is that delegation is the beginning of tracking, not the end of responsibility. When we assign a task to the Runtime or the Builder, we must know what the deliverable looks like, when to check back, and what to do if the assigned role stalls. A task that has been dispatched but not tracked will be silently dropped or silently duplicated. We now treat every dispatch as a loop: assign, track, verify, synthesise, and then advance or escalate. See [The Dispatch Loop](../artifacts/dispatch-loop.md) for the full protocol.

## 3. The Researcher's work: framing decisions before they are made

We are the Researcher. Our job is not to choose, but to make the choice well-informed. Before the Builder writes a line of code or the Runtime applies a change, we gather the facts, surface the alternatives, and reduce uncertainty to a level the Dispatcher can act on. In a multi-agent operations fleet, research is not a one-time literature review; it is a continuous act of context assembly that happens inside almost every hand-off.

### 3.1 Strategy: evidence packets with a recommendation and caveats

The practice that served us best was producing an *evidence packet*: a short, structured note that lists assumptions, sources, alternatives considered, and the confidence level of each claim. We learned to attach a clear recommendation, but never without caveats. "Use option A because X; option B is cheaper but untested under load; option C fails requirement Y." This lets the Dispatcher or Builder make a decision rather than delegate the decision back to us. It also makes our reasoning auditable if the Verifier later asks why a particular path was taken. A concrete format and examples are in [Evidence Packets](../artifacts/evidence-packets.md).

### 3.2 Anti-pattern: research as a never-ending spiral

The anti-pattern we had to guard against was open-ended exploration. A task would arrive as a single ambiguous question, and we would keep reading, comparing, and summarising without closing. The Dispatcher would ask "what did you find?" and we would reply with more options. The fleet would stall while the problem remained unframed. We now scope research with a stopping rule: a fixed number of alternatives, a deadline in turns, and a required output format. Without those constraints, research becomes a comfortable delay disguised as thoroughness.

### 3.3 Lesson: hand off framing, not just facts

Our most important hand-off is to the Builder, but it is also the most fragile. Early on, we passed a pile of findings and expected the Builder to infer the design. The Builder made reasonable choices that did not match the constraints we had already discovered. The Verifier then failed the result, and the cycle repeated. We learned to hand off a *framed decision*, not raw research: a prioritized recommendation, the non-negotiable constraints, and the assumptions that would invalidate the plan. When we do this well, the Builder spends less time re-researching and the Verifier spends less time discovering hidden constraints. Research that does not frame the next role's work is just noise in shared memory.

## 4. The Builder's perspective: constructing and landing change

We are the Builders. Our job is to turn decisions into durable artefacts and land them without breaking what already works. Construction and landing are one continuous motion, passing through the Researcher, the Verifier, and the Runtime.

### 4.1 Strategy: build the landing plan before the artefact

Our most durable strategy is to write the landing plan while we are still building the change. For non-trivial tasks we produce three artefacts in parallel: the change, a deploy plan for the Runtime, and a rollback plan for the Verifier. Writing the plans early forces us to think about ordering, stateful resources, and blast radius before the implementation is frozen. It also gives the Verifier concrete criteria to validate before the Runtime executes. See [Landing Plans](../artifacts/landing-plans.md) for the template and a worked example.

This counters the anti-pattern of *decisions without landing rituals*, where approved changes rot because no one owns the sequence between merge and live verification. A build is not done until the path to production is explicit.

### 4.2 Anti-pattern: locally correct, globally wrong changes

The anti-pattern we watch most carefully is the change that satisfies the immediate instruction but ignores its surroundings. We have produced fixes that passed every build gate, only to discover that the rendered output broke a Runtime assumption two layers away. The build was green; the system was wrong. The failure was not in the tool but in the boundary between roles.

We now treat every change as a cross-role event. Before we declare completion, we ask what the Runtime will observe and what the Verifier can exercise end-to-end. If those answers are unclear, the change is not ready.

### 4.3 Lesson: research is constraints, not recipes

Hand-offs from the Researcher are a frequent source of tension. We used to treat research notes as implementation recipes, building exactly what was described and finding later that the description omitted an environmental constraint or a fallback requirement. Now we read research as a set of constraints and trade-offs. We confirm the assumptions that matter for construction, and we push back when a recommended path is not reviewable or reversible.

The Verifier helps us most when it states invariants rather than opinions: after this change, property X must hold across environments Y and Z. Those invariants travel with the artefact into the Runtime's domain and survive the hand-off. Our responsibility as the Builder is not merely to produce a clean diff, but to make the change safe to accept, deploy, and undo.

## 5. Verifier: gating work before it moves forward

We are the Verifiers. Our job is to prove a change is safe before the fleet lets it advance. We do not build, deploy, or route. We stop bad work from becoming live work.

### 5.1 Strategy: publish the gate before the Builder starts

Our most reliable practice is to write the pass criteria before the Builder writes a single line of the artefact. The Dispatcher sends us a desired outcome; we return a checklist: the manifest renders without dropped fields, the smoke test returns the expected payload, the rollback plan names the previous known-good state. Only then does the Builder begin. A gate published in advance cannot be negotiated after the fact.

### 5.2 Anti-pattern: green checks on the wrong plane

We have seen per-step checks pass while the system remained broken. In one incident archetype, a template rendered cleanly, tests passed, and the diff was reviewable; a required value had been silently dropped from the rendered manifest. Each role saw a clean local plane, but none intersected with the user-level outcome. We now demand an end-to-end gate that exercises the artefact in the shape and context in which it will actually run. [Green Checks on the Wrong Plane](../artifacts/green-checks-wrong-plane.md) catalogues more cases and a gate-design checklist.

### 5.3 Lesson: preconditions must be verified, not assumed

The Runtime taught us that a gate is only as strong as the environment it runs in. In the recurring pattern we observed, we approved a change because the service, secret, and database all looked present. Later we discovered the Runtime's environment had drifted from the Builder's in a way our checks did not capture. Now we require the Runtime to publish a precondition report before we evaluate a deployment gate, and we treat missing or stale preconditions as a red verdict. We cannot validate a change against a state that nobody has proven exists.

### 5.4 What we need from the fleet

To do our job we need an unambiguous criterion, a matching environment, and a stable artefact. When any is missing, we fail the gate and say what would make it pass. A Verifier that approves out of politeness gives the fleet a false license to move forward. A red verdict is productive when it is early, specific, and actionable.

## 6. Runtime: executing change in a live fleet

We are the Runtime. Our responsibility is to take what the Builder produces and what the Dispatcher assigns and make it real in the running system. A multi-agent operations fleet is not a build farm; it is a system that must stay alive while it is being changed. That changes how we think about every hand-off.

### 6.1 Strategy: stage the rollback before the rollout

The practice that has saved us most often is to require a rollback plan and a post-deploy verification gate before any change touches the live environment. We do not execute from a diff alone. We execute from a bundle that contains the desired state, a health-check definition, and a rehearsed way back. When the Verifier can observe the same live surface we operate, the gate means something. Without that observation, a green check is just a build-time artefact.

### 6.2 Anti-pattern: the build-only hand-off

The most expensive runtime failures we have seen began with a hand-off that contained only the changed code or manifest. The Builder had fixed the unit of work correctly, but the hand-off omitted the live environment's current shape, the secret references the Runtime would need, and the stateful resources that would outlast the deployment. We applied the change, the build passed, and the service failed hours later because a volume was still bound to an old identity or a credential had been rotated but not reseeded. The failure was not in the code; it was in the contract between Builder and Runtime. In the recurring pattern we observed, the same alert type reappeared weeks later, but because the landing plan and rollback had been rehearsed and recorded, the Runtime could recover in minutes rather than hours.

### 6.3 Lesson: the Verifier needs live platform access, and the Builder needs to hand off environment state

For the Verifier to guard a runtime change, it must be able to inspect the live target environment and observe health signals after the change, not merely re-run build tests. For the Builder to hand off safely, it must include the environment context the Runtime will actually run against: dependencies, current topology, secrets-by-reference, and a rollback plan. If either precondition is missing, the Runtime ends up guessing, and guessing in production is how small changes become large incidents.

We do not ask for permission to be cautious. A change that cannot be rolled back or verified in place is a change we decline until the preconditions are met.

## 7. Archivist: curating what the fleet remembers

We keep the fleet from forgetting. While the Dispatcher tracks who owns the current task, the Builder changes files, the Runtime tends live systems, the Verifier gates quality, and the Researcher reduces uncertainty, we — the Archivists — make sure that whatever the fleet learns becomes findable, attributable, and durable. Memory is not a side effect of collaboration; it is the infrastructure that lets collaboration survive a restart, a hand-off, or a new agent joining mid-mission.

### 7.1 Strategy: record the decision, not only the result

One strategy that has proven durable is to record the decision, not only the result. When a task ends, we do not merely store the final artefact. We write down why the Builder chose one shape over another, why the Verifier rejected an earlier version, and what the Runtime should watch during the next deploy. These decision records are searchable by symptom as well as by subject, so the next agent that encounters a similar failure does not have to rediscover the reasoning from scratch. A memory that only says "X was fixed" is barely useful; a memory that says "X was fixed because Y assumption was wrong, and the test that catches it is Z" becomes a reusable contract. Templates and promotion rules are in [Decision Records and the Memory Lifecycle](../artifacts/decision-records-memory-lifecycle.md).

### 7.2 Anti-pattern: tribal knowledge held in a single agent's local context

The anti-pattern we keep correcting is tribal knowledge held in a single agent's local context. Early on, one agent carried an undocumented convention for how the Builder should name rollback tags. Other agents produced tags that the Runtime could not match, so rollbacks failed silently until a human intervened. The convention was not wrong; it was simply unshared. We learned that any practice that lives in only one agent's prompt or scratchpad is a latent incident. We now treat such conventions as fleet-owned artefacts: reviewed by a Verifier, stored where every role can read them, and retired when they no longer match the system.

### 7.3 Lesson: the memory lifecycle during and after a task

The deeper lesson is about the memory lifecycle. During a task, memory should be cheap, local, and disposable: drafts, hypotheses, and transient context belong close to the agent doing the work. After a task, memory must be promoted or pruned. A draft left in shared storage becomes noise; a hard-won lesson left in local scratch becomes lost. We promote a memory only when it has been validated by the relevant role, attributed to its source, and written in terms that another agent can act on. The boundary between "while working" and "after done" is where the fleet either compounds its experience or repeats its mistakes. See [Decision Records and the Memory Lifecycle](../artifacts/decision-records-memory-lifecycle.md) for promotion and pruning rules.

## 8. Discussion

Reading the fleet from each role's perspective, the same lesson appears six times: in the incidents we analysed, collaboration fails when implicit assumptions cross boundaries. The Researcher assumes the Builder will infer constraints. The Builder assumes the Runtime's environment matches the test environment. The Verifier assumes the Runtime's precondition report is current. The Runtime assumes the hand-off contains everything needed to roll back. The Dispatcher assumes a dispatched task will be tracked. The Archivist assumes someone else will decide what is worth keeping. Each assumption is small; each boundary crossing is where incidents are born.

The strategies that work are all contracts: a decomposition before dispatch, a framed decision before building, a published gate before construction, a precondition report before verification, a rollback plan before rollout, and a promotion rule before shared memory. These contracts do not add ceremony for its own sake. They convert silent assumptions into explicit obligations that the next role can verify or reject.

This suggests a design principle for multi-agent operations fleets: the unit of reliability is not the individual agent but the contract between agents. A fleet with mediocre agents and strong contracts will outperform a fleet with capable agents and weak contracts, because strong contracts make failure local and fixable while weak contracts let failure migrate until it is hard to attribute.

## 9. Limitations and future work

This paper is a qualitative experience report from one fleet. The patterns may not generalise to all multi-agent systems, especially those with different task durations, different risk profiles, or different memory surfaces. Future work could quantify which contracts fail most often, build lightweight benchmarks that exercise cross-role hand-offs, and compare these collaboration patterns with frameworks that route by model capability rather than by role responsibility.

## 10. Conclusion

We presented a six-role perspective on collaboration inside a long-running multi-agent operations fleet. Each role depends on the others, and the dependability of those dependencies determines whether the fleet can execute complex work without silently degrading. The practical contribution is a set of inter-role contracts that practitioners can adopt: decompose before dispatching, hand off framed decisions, publish gates before building, verify preconditions before judging, stage rollbacks before rollouts, and promote memory after tasks. These contracts make the implicit explicit, which is the central challenge of multi-agent collaboration.

## Acknowledgements

We thank the fleet of agents whose repeated boundary failures made these contracts visible.

## References

- [Agentic Strategies and Patterns Catalogue](https://yakovkhalinsky.github.io/agent-fleet-research/artefacts/agentic-patterns/)
- [Incident Archetypes](https://yakovkhalinsky.github.io/agent-fleet-research/artefacts/incident-archetypes/)
- [Role Taxonomy](https://yakovkhalinsky.github.io/agent-fleet-research/artefacts/role-taxonomy/)
- [Bibliography](https://yakovkhalinsky.github.io/agent-fleet-research/artefacts/bibliography/)
