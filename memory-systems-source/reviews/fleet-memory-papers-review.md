# Fleet Memory Papers Review

A multi-pass review of the fleet-memory papers (06–09) for tone, IP safety, language, and consistency.

## Final-Pass Review (06–09)

**Reviewer:** assistant
**Date:** 2026-07-06
**Scope:** Four fleet-memory papers before publication.

The four papers now largely match the site's established tone: first-person plural operational narration, pattern/archetype framing, anonymised examples using the six stylised roles, no product names, no first-person singular, and no American spellings.

## Earlier review notes

Earlier passes removed product names, first-person singular, American spellings, tool-name backticks, and lifted product-description language into pattern language. The major product-name scrub held across the final pass.

## Final-pass patches applied

| File | Change |
|---|---|
| `06-sync-taxonomy.md` | Abstract: standardised on `operation log (oplog)`. Comparison table: `always work` → `work offline`. Threats: `broadly applicable` → `more than a field-derived hypothesis`. `HTTPS` → `a secure transport`. References trimmed to a single line. |
| `07-local-first-runtime.md` | Title and headings changed from `gateway` to `runtime` / `federation` to avoid tool-name implication. Abstract, §4, and conclusion rewritten in pattern language. `sync agent` → `federation daemon`; `gateway process` → `network gateway`; `embedding worker` → `embedding queue`; `most` → `many`; `the bottleneck` → `bottlenecks`; `more valuable` → `more useful`. References already matched site style. |
| `08-explicit-identity.md` | `must never wait` → `must not wait`. `most common` → `more common`. References trimmed to a single line. |
| `09-federated-failure-modes.md` | `universal recipe` → `general recipe`. References trimmed to a single line; stub-paper `(planned)` note removed. `the moment of recovery is when pressure is highest` → `recovery usually happens under pressure`. |

## Verification checks

| Check | Result |
|---|---|
| Product / tool names in fleet-memory papers | None remaining (Raft/CRDT used only as generic design-contract examples; public framework names only in `08` related-work section). |
| First-person singular | None found. |
| American spellings | None found in the four papers. |
| Backticks / code-like formatting | None in the four papers. |
| Six stylised roles used in examples | Confirmed: Dispatcher, Builder, Runtime, Verifier, Researcher, Archivist. |
| Reference sections | All four now end with a single line: `See [fleet-memory bibliography](../artifacts/fleet-memory-bibliography.md).` |
| Anonymous examples | All incidents are anonymised archetypes; no hostnames, dates, project names, or identifiers. |
| Superlatives / overclaiming | Reduced to measured terms. |

## Publication verdict

**Publication-ready.** The four papers match the tone of the original field-study paper, respect the publishing-charter constraints, use consistent terminology with the fleet-memory patterns and archetypes, and present no remaining IP or language blockers.

## Remaining optional notes

- `06-sync-taxonomy.md` still uses `Raft` and `CRDT` as concrete examples of design contracts; this is acceptable because the paper explicitly compares design contracts, not implementations.
- `08-explicit-identity.md` §6 names public agent-memory frameworks. This is acceptable under the charter as related-work comparison, but authors may choose to soften or remove them.
- The bibliography itself (`fleet-memory-bibliography.md`) retains product URLs; this is a reference artefact, not a paper, and is consistent with academic citation practice.

**Recommendation:** Approve for publication after a quick author read-through of the patched lines.
