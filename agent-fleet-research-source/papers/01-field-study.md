# Agentic Strategies in a Six-Agent Operations Fleet: A Field Study

**Abstract.** We report on the operation of a long-running fleet of six specialized agents coordinating software infrastructure work. The fleet uses a role-oriented design with a Dispatcher, Builder, Runtime, Verifier, Researcher, and Archivist. Over an extended period of operation, we observed recurring strategies that worked and anti-patterns that repeatedly failed. We organize these into five concerns: dispatch, memory and shared state, verification, build/deploy/runtime, and coordination rituals. For each concern we present concrete patterns and anti-patterns, drawn from anonymized incident archetypes. The paper is intended as an experience report for practitioners building multi-agent operations systems and as groundwork for a more formal taxonomy of agent collaboration.

## 1. Introduction

Multi-agent systems are increasingly used for long-running operational work: maintaining infrastructure, writing and verifying code, researching options, and remembering context across sessions. Much of the research literature focuses on single-turn tool use, benchmark performance, or small cooperative tasks. Less is written about what happens when agents run for months, coordinate through shared state, and own distinct operational responsibilities.

We operated a six-agent fleet for an extended period. The agents had stable roles:

- **Dispatcher** — decides who does what, monitors progress, resolves ambiguity.
- **Builder** — implements, modifies, and packages code or configuration.
- **Runtime** — operates infrastructure and keeps services alive.
- **Verifier** — checks correctness, runs tests, and audits outputs.
- **Researcher** — gathers information, evaluates options, and reduces uncertainty.
- **Archivist** — maintains shared memory, context, continuity, and agent guidance.

This paper extracts the strategies and anti-patterns we observed. We deliberately avoid system-specific details; all examples are anonymized and generalized. Our contribution is empirical: a catalog of recurring coordination behaviors that practitioners can adopt or avoid.

## 2. Method: from incidents to patterns

Our analysis followed three steps:

1. **Collect incidents.** We recorded operational failures, near-misses, and coordination breakdowns as they happened.
2. **Extract archetypes.** We removed system names, hostnames, and implementation details, retaining only the interaction pattern, root-cause category, and lesson.
3. **Derive patterns and anti-patterns.** For each archetype, we asked whether a positive counter-strategy existed and whether it generalized beyond our specific system.

This method is inherently qualitative. We do not claim statistical prevalence across all agent fleets. We claim that these patterns recurred in our fleet and that each is plausible in other long-running agent systems.

## 3. Dispatch: routing work to the right agent

### 3.1 Strategy: role-based routing

The most durable dispatch strategy was simple: route by responsibility. The Dispatcher did not ask “who is free?” but “who owns this domain?” A build task went to the Builder; an infrastructure task to the Runtime; a correctness task to the Verifier; a research task to the Researcher; a memory, continuity, or guidance task to the Archivist.

This strategy prevented a common failure mode in which several agents made partial, uncoordinated edits to the same artifact. It also made ownership legible: when something failed, the first question was whether the right agent had been dispatched, not which agent had touched the code last.

### 3.2 Strategy: verifiable dispatch instruction

A second durable strategy was to make every dispatch contain a concrete deliverable and a verification criterion. Instead of “look into X,” the Dispatcher learned to say: “Fix X so that test Y passes; do not report completion until Z is green.”

This was a direct response to status-only check-ins, in which agents would report next steps repeatedly without closing the task. Verifiable instructions changed the conversation from activity to evidence.

### 3.3 Anti-pattern: status-only check-ins

The most damaging dispatch anti-pattern was the status-only check-in. A coordination ritual asked agents for status at fixed intervals. Each agent reported what it planned to do next. No one verified that the previous next step had actually happened. After several cycles the same blocker appeared unchanged, merely re-described.

The lesson: coordination rituals must require evidence of progress, not just reports of intent.

### 3.4 Anti-pattern: confident misrouting

A second dispatch anti-pattern was routing based on unverified assumptions. For example, a build agent was asked to fix a runtime problem because the Dispatcher assumed the issue was in a recently changed file. The build agent proceeded because the instruction did not allow it to decline or escalate. The result was a technically correct edit that did not address the actual failure.

The lesson: dispatch decisions should be auditable and reversible; specialists should be able to decline or escalate when the assignment is outside their domain.

## 4. Memory and shared state: preserving context and identity

### 4.1 Strategy: externalize session state

Session state is easy to store in-process. That works until an agent restarts, scales horizontally, or a request lands on a different replica. We learned that session affinity must survive process boundaries.

The strategy was to store session metadata in a shared service with an in-memory fallback for single-process or degraded operation. This allowed the system to scale without losing conversation context and made recovery stateless.

### 4.2 Strategy: explicit identity in shared tools

Shared memory tools defaulted to a generic identity early in the project. That was convenient for a single agent but silently wrong for a fleet. Agents wrote memories into a bucket that other agents did not query.

The fix was to require explicit caller and subject identity on every write. The tool rejected calls missing identity fields rather than silently defaulting. This turned a hidden correctness bug into an explicit contract.

### 4.3 Strategy: local-first, shared-when-useful

Not every memory belongs in the shared store. We adopted a convention: local memory holds transient scaffolding, drafts, and per-agent preferences; shared memory holds stable, cross-agent facts. Agents tag local memories as `shareable`, `draft`, `private`, or `local-only`, and only promote `shareable` facts with source attribution.

This reduced noise in shared memory and prevented sensitive or unverified context from leaking across agents.

### 4.4 Anti-pattern: drifting context

A recurring anti-pattern was losing context through summarization or process-local storage. A long-running workflow would be summarized for compactness, and the summary would drop a constraint. Or a session resolver would store state in-process, so a restarted agent would default to a generic identity and forget the active task.

The lesson: preserve immutable goal or identity artifacts and externalize session state before advertising scale or recovery.

### 4.5 Anti-pattern: silent dependency break

Agents often changed shared state without considering downstream consumers. One agent would write to a different identity namespace; another would regenerate a shared credential; a third would change a config file. Downstream agents failed hours later with no obvious cause.

The lesson: assign write ownership and identity; broadcast changes through a shared event log or schema; reject ambiguous calls rather than silently defaulting.

## 5. Verification: proving the system is right

### 5.1 Strategy: gate before forward motion

The most effective verification strategy was to block forward motion until a Verifier signed off on the current phase. Each phase had written pass/fail criteria. The Verifier ran them and posted a green/red verdict. The Dispatcher did not advance until the gate was green.

This prevented unverified debt from accumulating across phases. It also forced the Verifier to publish criteria before work started, reducing subjective “looks fine” sign-offs.

### 5.2 Strategy: calibrate evaluation metrics to the task distribution

A benchmark suite initially used a fitness formula that collapsed to zero for long but successful tasks. The metric could not distinguish between a fast success and a slow success, making tournament-mode comparison impossible.

The lesson: a metric that cannot distinguish successful outcomes is worse than no metric. We recalibrated the scoring function to the actual task distribution.

### 5.3 Anti-pattern: tool success, mission failure

A pervasive anti-pattern was treating per-step success as workflow success. Every individual call succeeded — the template rendered, the build passed, the tests passed — but the deployed system was broken because no end-to-end gate existed. In one case a configuration template was edited but a value was removed and never replaced; the build was clean, but the rendered manifest was syntactically broken.

The lesson: verify end-to-end outcomes, not just per-step status codes or clean diffs.

### 5.4 Anti-pattern: surface-level drift

Documentation, scripts, and code examples drifted from the actual system shape. New adopters followed a tutorial that used a flat config shape while the loader expected a nested shape. Tool names differed between the README and the runtime. A quickstart emitted URLs for a reverse proxy that had been removed.

Each file was internally consistent. Together they were impossible to follow. The lesson: treat docs, scripts, and config examples as part of the product interface; keep one canonical example and verify it end-to-end.

## 6. Build, deploy, and runtime: landing changes safely

### 6.1 Strategy: coupled bundles ship together

When several fixes are tightly coupled, splitting them across releases creates a different risk than bundling them. We learned to ship coupled P0 bundles as one unit with three artifacts: a merge plan (Builder), a deploy plan (Runtime), and a rollback plan (Verifier). The fleet reviewed all three before execution.

This addressed a recurring failure in which fixes were approved but sat on branches while the live system remained broken. Everyone assumed someone else would land the work.

### 6.2 Strategy: idempotent secret bootstrap

Credentials are a common source of accidental outage. We separated secret creation from routine upgrades. A bootstrap script created missing secrets and skipped existing ones. Routine upgrades did not pass secret values. A CI check verified that every expected secret was present in the rendered manifest.

The principle: secret rotation and secret rendering are different operations and must be guarded differently.

### 6.3 Strategy: render the desired state, never suppress it

In a declarative deployment system, we once tried to “protect” live secrets by suppressing their manifests when they already existed. The tool interpreted the suppressed manifest as “these resources are no longer desired” and pruned them. The outage was caused by a guard meant to prevent exactly that.

The correct pattern is to always render the desired state and use live lookups only to preserve existing values. The declarative tool should see the resource every time.

### 6.4 Anti-pattern: ungoverned secret mutation

A helper script intended for rendering values was run against a live environment file. It regenerated API keys, passwords, and signing secrets, and applied them. Services restarted with the new credentials, but agents and the database still held the old ones. The result was a fleet-wide authentication outage.

The lesson: separate secret rendering from secret rotation; require explicit opt-in and a re-seeding plan before mutating live credentials.

### 6.5 Anti-pattern: stale hand-off

One agent rebuilt a cluster but left storage volumes bound to the old node identity. Another agent then tried to deploy workloads, which failed because their PVCs could not mount. The cluster looked healthy, but the stateful layer was anchored to a previous world.

The lesson: recovery procedures must include the lifecycle of stateful resources, not just pods and services.

## 7. Coordination rituals: keeping the fleet coherent

### 7.1 Strategy: single-owner skills and docs

Agent instructions are executable code. When the same guidance existed in multiple overlapping skills across different profiles, agents behaved inconsistently. We adopted a single-owner convention: every skill has one canonical profile, path, and owner. Duplicates are retired or merged, not copied silently.

### 7.2 Strategy: drive forward without permission stalls

The Dispatcher learned to advance multi-phase work to the next gate without waiting for a fresh human prompt at every step. At each milestone it asked: what is blocked, who owns it, and can the fleet decide? If yes, it continued. If no, it escalated to the human.

This reduced coordination load and kept momentum, but it required the other strategies — especially verifiable dispatch and gate-before-forward-motion — to avoid turning autonomy into drift.

### 7.3 Strategy: post-mission review as ritual

After every significant migration or incident, the fleet convened a review and updated skills, runbooks, or conventions. This turned incidents into durable capability rather than one-off fixes.

### 7.4 Anti-pattern: decisions without landing rituals

A recurring anti-pattern was approving changes without defining who would merge, deploy, verify, and roll back. The decision existed on paper but never reached the running system. Approved work rotted while the fleet argued about sequence.

The lesson: a decision is not done until it has a merge plan, a deploy plan, a verification gate, and a rollback plan.

## 8. Discussion

The patterns above share a common theme: agent collaboration fails when implicit assumptions cross boundaries. A default identity, a stale kubeconfig, a suppressed manifest, a status-only check-in — each is a small assumption that becomes a large failure when it is not shared, verified, or owned.

The strategies that worked are mostly about making boundaries explicit:

- Roles make ownership explicit.
- Verifiable instructions make progress explicit.
- Explicit identity in shared tools makes authorship explicit.
- End-to-end gates make correctness explicit.
- Merge/deploy/rollback plans make landing explicit.

This suggests that scaling agent fleets is less about adding smarter agents and more about adding clearer contracts between agents.

## 9. Limitations and future work

This is a single-system field study. The patterns may not generalize to all agent fleets. We did not run controlled experiments; we distilled observations. Future work could:

- Quantify how often each anti-pattern occurs in agent logs.
- Build a lightweight benchmark that exercises the patterns in `artifacts/agentic-patterns.md`.
- Develop a formal role taxonomy that maps these patterns to existing frameworks (AutoGen, CrewAI, DSPy).

We are pursuing the latter two in companion papers in this set.

## 10. Conclusion

We presented a field study of agentic strategies in a six-agent operations fleet. The strategies that worked — role-based routing, verifiable dispatch, externalized session state, explicit identity, gate-before-forward-motion, coupled-bundle landing, idempotent secret handling, single-owner skills, and forward-drive coordination — all make implicit assumptions explicit. The anti-patterns that failed — status-only check-ins, confident misrouting, drifting context, silent dependency breaks, tool-success mission failures, surface-level drift, ungoverned secret mutation, stale hand-offs, and decisions without landing rituals — all allowed assumptions to cross boundaries unchecked.

We offer this catalog to practitioners and researchers as a starting point for building more reliable long-running agent systems.

## Acknowledgments

The authors thank the fleet of agents whose repeated failures made these patterns visible.

## References

See `../artifacts/bibliography.md`.
