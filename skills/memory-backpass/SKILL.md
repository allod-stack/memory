---
name: memory-backpass
description: Run weekly on a timer or by hand to read recent session traces against the memory index and open one PR of evidence-backed edits.
---

# Memory Backpass

Read recent distilled session traces against the public and private memory indexes, preserve quote-checked sightings in their ledgers, and propose one small human-reviewed update to private memory. The first two runs are report-only by the owner's decision and must be compared by hand before any timer is created.

This procedure names no provider or model. Read the deployment's `agent-roster` skill to choose its current bulk-analysis model and strongest synthesis model; never substitute another model silently.

## Inputs

Read these inputs in order:

1. The traces checkout: a git repository of distilled session traces produced by `allod trace distill`, with one Markdown file per session at `<machine>/<harness>/<date>-<session-id>.md`.
2. The public memory checkout.
3. The private memory checkout, `agent-memory`.
4. `<traces>/.memory-backpass/analyzed.txt`, one session-file path per line. Create it if absent. Commit it to the traces repository at the end of a successful run.

Accept `--report-only`. Keep an owner-controlled count of completed report-only runs and do not schedule this skill until the owner has compared the first two reports.

## Procedure

1. Pull the traces checkout and both memory repositories. Number every bullet in each `memory.md` by current position: `PUB-001`, `PUB-002`, ... for public memory and `PRV-001`, `PRV-002`, ... for private memory. Render each as `[id] text`; the ids are run-local references, not text written back to memory. Render every open line from both `ledger.md` files with the stable id `L-<first 8 hex characters of the line's SHA-256>`.

2. For each trace path absent from `analyzed.txt`, make one call to the deployment's bulk-analysis model with [analysis-prompt.md](analysis-prompt.md), filling every `{{NAME}}` placeholder with the instruction index, open ledger entries, trace path, and trace. Gate success on the returned content, never the process exit status. The answer must parse as one JSON object containing `positive`, `negative`, and `gaps` arrays in the prompt's schema. Retry an empty or unparseable answer once with the same model. After a second failure, record that trace as failed in the run report; do not silently change models and do not mark it analyzed.

3. Mechanically whitespace-fold each returned `quote` and the trace, then discard every item whose folded quote is not a substring of the folded trace. This check is mandatory even when the model reports high confidence. Count discarded items for the funnel.

4. For each surviving gap or negative item, choose the owning repository by `memory.md`'s "Public vs Private Memory" rule; when unclear, choose private. Prepare one ledger line per item and session: `gap` for a gap, `ignored` for `non-compliance`, and `harm` for `harm`. Do not ledger `irrelevant` items. For `ignored`, name the existing rule by its opening words. Never add a second line for the same rule and session. Public-memory ledger additions are relay proposals, because this private deployment cannot push public; private-memory additions belong in the private change worktree created below. Add each successfully analyzed trace path to the state-file update.

5. Fold the surviving evidence. For each instruction id, count positive, non-compliance, and harm sightings and the distinct sessions behind each. Its relevance is the share of all successfully analyzed sessions in which it drew at least one item. For each ledger rule, count distinct source sessions, treating entries judged to express the same rule as one corroboration group.

6. Prepare both ledger prunes: remove a sighting when the date encoded in its trace filename is more than ninety days old, or when its proposed rule now appears in a topic file or either index. Use literal substring coverage when sufficient and the bulk-analysis model's judgment on the fold when meaning, rather than exact wording, establishes coverage. Public prunes remain relay proposals; apply private prunes only in the private change worktree.

7. Unless `--report-only` is active, create a private-memory worktree with `allod change begin -d memory-backpass-<date> agent-memory`. Apply the prepared private ledger changes there. Make one call to the deployment's strongest model with [synthesis-prompt.md](synthesis-prompt.md), filling in the fold; both numbered indexes and their byte counts against the caps read from `.hooks/memory-budget`; topic filenames with their index descriptions; and titles of closed, unmerged private-memory PRs beginning `memory-backpass:`, read with `forge pr list -s closed`. Give the model the private worktree and require it to edit there. It may propose public edits only under `To relay`; it must never edit the public checkout.

8. Run `.hooks/memory-budget --worktree` in the private worktree. Give a failure back to the same synthesis model to fix within this run. If the corrected edit still fails, drop that edit. Do not raise a cap.

9. Run `allod change record` and `allod change submit` from the private worktree. Use title `memory-backpass: <date>`. The body starts with one plain paragraph giving sessions read, findings kept and quote-dropped, and edits proposed. Follow it with one section per edit containing its title, diff, rationale, and verbatim quotes with `<harness> <session id>` sources; then `To relay`; then a funnel counting traces analyzed, items returned, items dropped for a missing quote, ledger entries corroborated, and edits proposed. Every memory write remains behind this pull-request review; never push a memory repository's default branch and never merge.

10. With `--report-only`, do everything except steps 7 through 9. Print the fold and every would-be ledger addition or prune, and write nothing to either memory repository. Still commit the successful trace-path additions to `analyzed.txt` in the traces repository at the end.

## Synthesis gates

Enforce these rules in the synthesis prompt and when reviewing its result:

- Propose at most five edits.
- An add, rewrite, or removal of an index line needs supporting quotes from two distinct sessions.
- A removal needs `harm` sightings from two distinct sessions. Several `non-compliance` sightings call for reinforcement, earlier placement, or a sharper trigger, never removal.
- Every edit includes at least one verbatim quote and its source.
- The post-edit index fits the cap in `.hooks/memory-budget`.
- A rule relevant to fewer than one fifth of analyzed sessions belongs in a topic file rather than the index.

## Self-exclusion

Every prompt sent by this skill must begin with `<!-- memory-backpass:self -->`, as both prompt files do. `allod trace distill` drops a session whose first user message starts with that line, so this procedure never analyzes its own model calls. Preserve the marker when retrying or adding diagnostic text to a prompt.

## Failures

Stop with a one-line report when the traces checkout is missing, a required model is unreachable, or a credential is refused. Never fall back to another model silently. A run never pushes to a memory repository's default branch and never merges.
