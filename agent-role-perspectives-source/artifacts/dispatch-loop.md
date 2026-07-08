# The Dispatch Loop: From Assignment to Advancement

This page expands the Dispatcher lesson in *Working Together*: delegation is not a hand-off. The Dispatcher's loop is the operational heartbeat of the fleet. When it runs cleanly, work moves; when it stalls, the whole fleet feels it.

## The loop

The Dispatcher repeats five steps for every task:

1. **Assign.** Name the role, the deliverable, and the verification criterion.
2. **Track.** Know when the deliverable is due and what a stall looks like.
3. **Verify.** Confirm the deliverable against the criterion, not against activity.
4. **Synthesise.** Combine the result with any parallel work and state the next decision.
5. **Advance or escalate.** If the fleet can decide, continue. If not, raise a true blocker.

## Assignment checklist

A good assignment answers:

- **Who owns it?** One role. Not "the fleet" or "someone."
- **What is produced?** A file, a verdict, a plan, a deployed state.
- **How will we know it is done?** A test passes, a checklist is green, a metric is reported.
- **What is out of scope?** Boundaries prevent another role from duplicating the work.
- **When should we check back?** A time or a signal, not "soon."

## Tracking signals

A task is stalled when any of the following is true:

- The owner has not produced evidence of progress within the expected window.
- The owner reports a blocker that has not been assigned to the right role.
- The same status is reported more than once without a change in evidence.
- A parallel dependency has not been acknowledged by its owner.

## Reclaim or re-dispatch

When a task stalls, the Dispatcher has three options:

1. **Prompt the owner** with sharper context and a deadline.
2. **Re-dispatch with a narrower scope** if the original task was too broad.
3. **Reclaim and act directly** only when data loss or outage is imminent, then hand verification back to the original owner.

The third option is the exception. If it becomes common, the fleet has a dispatch-trust problem, not a task problem.

## Example: a stalled verification

The Dispatcher assigns the Verifier a live acceptance gate. The Verifier returns a red verdict because a precondition is missing. The Dispatcher does not ask the user what to do; it routes the precondition gap to the Runtime, confirms the fix, and re-runs the gate. If the Runtime cannot fix it, the Dispatcher escalates the specific resource gap, not the vague task.

## Example: an incident hand-off

During a live incident, the Dispatcher receives a stream of symptoms from monitoring. It gives the Researcher a bounded question — "Has this alert pattern appeared before, and what was the last known good configuration?" — and sets a ten-minute deadline. The Researcher returns an evidence packet naming the rollback target. The Dispatcher routes the rollback to the Runtime with an explicit verification gate, then tells the Archivist to record the decision once the service recovers. The Dispatcher’s value is not solving the incident; it is preventing the incident from being solved twice in parallel, or not at all.

## Why this deserves its own page

The main paper states the loop in one paragraph. In practice, the loop is a protocol that every Dispatcher must run consistently. A standalone page lets practitioners adopt it as a runnable template and gives the Dispatcher a reference to point other roles to.
