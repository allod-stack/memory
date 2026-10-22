---
name: memory-backpass
summary: Turn session traces into one PR of evidence-backed memory edits
description: Run `allod memory backpass` to read recent session traces against the memory indexes and open one PR of evidence-backed edits, then review what it opened. Nightly on a timer or by hand.
---

# Memory Backpass

The backpass reads distilled session traces against the public and private memory indexes, keeps the sightings that survive a quote check, preserves them in the private ledger, and proposes one small human-reviewed update to private memory as a pull request. It is a program, `allod memory backpass` in `allod/tools`; this skill says how to run it and what to do with its output. The command's full contract, the resolution of its inputs, the seat contract, the state file, the exit codes and the report shape, is `docs/allod-memory.md` in that repository.

The two prompt files beside this skill, `analysis-prompt.md` and `synthesis-prompt.md`, are what the command sends to the models; it reads them from this checkout at run time, so an edit here changes the next run without a rebuild. Both must keep `<!-- memory-backpass:self -->` as their first line (Self-exclusion, below), and the command refuses a template whose placeholders it does not know.

## Running it

On a machine whose checkouts follow the registry convention, the bare command finds everything:

    allod memory backpass --report-only
    allod memory backpass

Report-only prints the fold and every ledger line and prune it would write, and writes nothing to either memory repository; it still records which traces it read. Without the flag it opens the PR. The report's first section names each input it resolved and where it came from; the `## Seats` section names the model that analyzed each trace; the funnel is the alarm: a quote-drop rate far above a few percent means corrupted input, not a bad model. Exit 3 means at least one trace failed every attempt and stays pending for the next run.

Model choice is the deployment's `agent-roster` skill's. With no seat given the command runs pi with its own configured default and says so in the report; `--bulk-seat`, `--fallback-seat` and `--synthesis-seat` take a wrapper script per model. It never substitutes a seat silently.

## After a run

- Review the PR titled `memory-backpass: <date>` on the private memory repository. Each edit section carries its rationale, its quotes with `<harness> <session id>` sources, and an evidence count; `one-session exception` marks an edit that rests on one session and must state its safety reason.
- `To relay` lists any public branch the run prepared, with its `allod patch receive` line; it is left unpushed for a human. A merged run whose relay line is still unchecked is listed at the top of the next report until the public half lands.
- Merge or close; the command never merges and never pushes a default branch.

## Step map

The command's code cites these step numbers; the mechanism behind each is in `docs/allod-memory.md`.

1. Number the index bullets `PUB-nnn`/`PRV-nnn`, id open ledger lines `L-<hash>`, list merged runs with an unrelayed public edit.
2. Analyze each pending trace with the bulk seat: one retry, then the fallback seat; gated on parsed content, never exit status.
3. Whitespace-fold every quote against its trace; keep substrings only.
4. One private ledger line per surviving rule and session; corroborate an existing line instead of duplicating it; never ledger an `irrelevant` item.
5. Fold the evidence: per-instruction counts and relevance, per-rule session counts, every surviving quote.
6. Prune pre-run ledger lines over ninety days old or now covered by a topic file; a public line is only proposed for relay.
7. Private clone, ledger commit, synthesis call; public proposals as commits in a second unpushed clone.
8. Byte budget: one fix call, then every proposed edit is dropped; caps are never raised.
9. `allod change record` and `submit`; label `relay` when the checklist is non-empty.
10. Report-only: everything but 7 to 9.

## Synthesis gates

These are the synthesis prompt's hard rules; the command reports them per edit for the human and does not revert a model's edit itself:

- At most five edits.
- An add, rewrite, or removal of an index line needs quotes from two distinct sessions; the only exception is a one-session add that states its safety reason and is flagged in the PR body.
- A removal needs `harm` sightings from two distinct sessions; `non-compliance` calls for reinforcement, earlier placement, or a sharper trigger, never removal.
- Every edit includes at least one verbatim quote and its source, drawn from the fold.
- The post-edit index fits the cap in `.hooks/memory-budget`.
- A rule relevant to fewer than one fifth of analyzed sessions belongs in a topic file, not the index.

## Self-exclusion

Every prompt the command sends begins with `<!-- memory-backpass:self -->`, as both prompt files do. `allod trace distill` drops a session whose first user message starts with that line, so the pass never analyzes its own model calls. Keep the marker when editing either prompt.
