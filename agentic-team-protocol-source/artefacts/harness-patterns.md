# Harness Patterns

This artefact expands Section 7 of [A Protocol for Role-Based Agent Teams](../papers/01-protocol.md) into practical wiring for three existing harnesses: pi.dev, Claude, and Cursor.

## Why this deserves its own page

Practitioners do not need a new framework; they need a way to apply the protocol with the tools they already have. This page gives concrete file layouts and wiring rules for each harness.

## Common primitives

Regardless of harness, every instantiation must preserve these primitives:

| Primitive | Purpose |
|---|---|
| `Package` definition | Declares the team shape, roles, records, and state machine. |
| Role identity | `agent:{role}:{instance_id}` as the observer/actor identity. |
| Durable records | `run_log`, `goal_record`, `dispatch_instruction`, `verdict`, `escalation_record`. |
| Tool allow-list per role | Only the tools relevant to the active role are exposed. |
| Verifier gate | A green/red verdict that advances the state machine. |

## pi.dev

- **Role representation:** prompt templates in `roles/{role}.md` inside a Pi package.
- **Package:** a Pi package directory with `pi.toml`, `package.yaml`, and role/task/tool files.
- **Routing:** an external Role Router selects the next role and invokes Pi with the matching prompt and tool set.
- **Memory:** all durable state lives in Eden. Pi's internal session state is intentionally bypassed.
- **Tool filtering:** Pi manifest declares per-role tools; the runtime also enforces an allow-list.
- **Verification:** a Verifier Pi agent writes an Eden `verdict` conclusion.

## Claude

- **Role representation:** Claude Project instructions, either as sections in one project or separate Projects per role.
- **Package:** a Claude Project with attached knowledge files and MCP servers.
- **Routing:** an external controller decides which Project to prompt next and propagates the package instance session id.
- **Memory:** all package state lives in Eden; Claude conversation context is treated as a scratchpad.
- **Tool filtering:** per-Project MCP config plus runtime-level allow-list.
- **Verification:** a Verifier Project writes a `verdict` conclusion to Eden and optionally produces a Claude Artifact for human review.

## Cursor

- **Role representation:** `.cursorrules` root file plus per-role `.cursor/rules/{role}.mdc` files.
- **Package:** a Cursor workspace with `.cursor/cells-config.yaml` for instance metadata.
- **Routing:** user `@` mention, root `.cursorrules` router, or external controller.
- **Memory:** all package state lives in Eden; local chat history is discarded after each turn.
- **Tool filtering:** `.cursor/mcp.json` plus role rule allow-list plus optional external filter.
- **Verification:** a Verifier rule writes an Eden `verdict` and may leave a repo marker file for human visibility.

## Cross-harness comparison

| Concern | pi.dev | Claude | Cursor |
|---|---|---|---|
| Role representation | `roles/{role}.md` | Project instructions / separate Projects | `.cursor/rules/{role}.mdc` |
| Package | Pi package | Claude Project | Workspace + instance config |
| Routing | External router | External controller | User `@`, root rules, or controller |
| Memory | Eden conclusions | Eden conclusions | Eden conclusions |
| Tool filtering | Pi manifest + allow-list | Per-project MCP + allow-list | `mcp.json` + role rule + filter |
| Verification | Verdict conclusion | Verdict + Artifact | Verdict + repo marker |
| Best for | Self-hosted, terminal-first | Human review, rich artefacts | Code-heavy, editor-centric |

## Implementation checklist

- [ ] Each role has a dedicated prompt/rule/instruction file.
- [ ] Roles are mapped to identity metadata on every Eden write.
- [ ] The runtime queries Eden at the start of every turn.
- [ ] Each role receives only its allowed tools.
- [ ] Verifier gates are stored as Eden `verdict` conclusions.
- [ ] All package state survives a process/window restart because it lives in Eden.

## Starter templates

Concrete starter templates for all three harnesses are available in the companion repository:

- [github.com/yakovkhalinsky/eden-cells/templates](https://github.com/yakovkhalinsky/eden-cells/tree/main/templates)
