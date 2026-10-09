---
name: memory-prune
summary: Drop index lines a window of sessions shows never changed an outcome
description: Score every index line against recent sessions and retire the ones that never steered one. Use when an index has grown and the question is which lines are still earning their bytes; run by hand after a `memory-backpass` run. This pass only removes, and only from the two `memory.md` indexes — additions, rewordings, topic files and the ledger belong to `memory-backpass`.
---

# Memory Prune

Measure every line of the public and private `memory.md` against a window of recent sessions, and retire the lines that window shows never changed what an agent did. Every removal is appended verbatim to `retired.md` first, so any line can be put back from the repository without reconstructing it.

This pass is subtractive and, after scoring, mechanical: it proposes no new text and rewords nothing, so it needs no synthesis model. Reinforcing a line that is being ignored, sharpening its trigger, moving it nearer the top, promoting a ledger rule and every edit to a topic file remain `memory-backpass`'s work. Running both over the same traces is expected; they answer different questions and neither reads the other's state.

This procedure names no provider or model. Read the deployment's `agent-roster` skill to choose its current bulk-analysis model and the fallback that skill names; never substitute another model silently.

## Inputs

Read these inputs in order:

1. One or more traces checkouts, each a git repository of distilled session traces produced by `allod trace distill`, with one Markdown file per session at `<machine>/<harness>/<date>-<session-id>.md`.
2. The public memory checkout.
3. The private memory checkout, `agent-memory`.
4. `retired.md` in each memory repository, which lists what earlier runs removed. Create it if absent, with the header its format block below describes.
5. `<traces>/.memory-prune/scoreboard.tsv` in each traces checkout, the previous run's score for every line, one `I-<hash>` per row. Create it if absent. Commit the replacement to its traces repository at the end of a successful run.

Accept `--window <n>` (default 60), `--report-only`, `--prune-topic-pointers`, and `--prune-unscoreable`.

## Procedure

1. Pull every traces checkout and both memory repositories. Number every bullet in each `memory.md` by current position, `PUB-001` upward for public memory and `PRV-001` upward for private memory, and render each as `[id] text`. The ids are run-local references for the scoring prompt and are never written into a memory file or into `retired.md`.

2. Give every line a stable identity independent of its position: `I-` followed by the first eight hex characters of the SHA-256 of its whitespace-folded text. A line that has been reworded is a new identity, which is correct — a rewritten line has not yet had its chance to steer anything.

3. Choose the window. Sort every trace file in every checkout by the date in its filename and take the newest `--window` sessions. Record the oldest and newest dates in it; they bound every claim this run makes.

4. For each trace in the window, make one call to the deployment's bulk-analysis model with [score-prompt.md](score-prompt.md), filling every `{{NAME}}` placeholder. Gate success on the returned content, never the process exit status. The answer must parse as one JSON object containing a `sightings` array in the prompt's schema. Retry an empty or unparseable answer once with the same model, then make one attempt with the fallback model the roster names for this skill, and record which model scored each trace in the run report. A trace that fails the fallback attempt too is recorded as failed and is not counted in the window.

5. Whitespace-fold each returned `quote` and the trace, and discard every sighting whose folded quote is not a substring of the folded trace. This check is mandatory however confident the model reports itself. Count the discards for the funnel.

6. Stop if fewer than three quarters of the window scored successfully. Report the scores and propose nothing: a thin window cannot distinguish a line nothing needed from a line nothing was asked about.

7. Score each line over the window: the distinct sessions in which it was `steered`, `ignored` and `harm`, and the share of scored sessions in which it drew any sighting at all. Classify it:

   - **live** — at least one `steered` or one `harm` session. It changed an outcome. Keep it.
   - **inert** — at least one `ignored` session and no `steered` or `harm` session. It came up and changed nothing.
   - **unused** — no sighting of any kind. Nothing in the window touched what it governs.

8. Require stability before removing anything. `git blame` each candidate line in its index and read the commit date; a line whose commit is newer than the oldest trace in the window is not eligible, because part of the window predates it. Report it as `too new` with the date that excluded it.

9. Hold two classes of line back from removal whatever they scored, and name every override in the report:

   - A line in a Topic Files section that names a topic file present in the repository. The pointer and its one-line description are the whole discovery mechanism for that file, so the pointer's disuse measures the topic, not the line; removing it orphans a file that is still correct. `--prune-topic-pointers` removes the guard.
   - A line in a Memory File Hygiene section. Its audience is this pass and `memory-backpass`, whose own sessions every trace window excludes by the self-marker below, so it cannot draw a sighting however well it is working. It is unscoreable, not unused. `--prune-unscoreable` removes the guard.

   These are the two shapes whose disuse is known to mean something other than disuse. A line held back for either reason stays in the report every run; the guard is a stated default, not a verdict.

10. Decide. An eligible `unused` line is retired this run. An eligible `inert` line is retired only when the previous run's `scoreboard.tsv` scored the same `I-` identity `inert` as well; on its first window it is reported as inert and left alone, so that a line failing to steer gets one `memory-backpass` cycle to be reinforced or sharpened before it is dropped. Propose at most five removals in one run, worst-scoring first, and report the rest as next run's candidates.

11. Append every removal to `retired.md` in its owning repository before taking the line out of the index, newest block first, in this format:

        ## I-<hash> <oldest trace date>..<newest trace date>

        <the removed line, verbatim>

        <section it came from> | <class> | steered <n> ignored <n> harm <n> of <m> scored sessions | <branch that removed it>

    Never remove a line whose identity already has a block in `retired.md`: an earlier run retired it and something has put it back, which is a decision to look at rather than to repeat.

12. Unless `--report-only` is active, apply the private removals in a worktree created with `allod change begin -d memory-prune-<date> agent-memory`, run `.hooks/memory-budget --worktree` there, then `allod change record` and `allod change submit` with title `memory-prune: <date>`. Apply the public removals in a worktree created with `allod change begin -d memory-prune-<date> allod/memory`, as plain commits carrying the score in the commit message, run the hook there too, and leave the branch unpushed; hand it to the human as the one line `allod patch receive <this machine>:<worktree path> allod/memory`. Never push a memory repository's default branch and never merge.

13. The pull-request body opens with one plain paragraph giving the window, how many sessions scored, how many lines were measured, and how many are proposed for removal. Follow it with one section per removal carrying the line, its score, its class, and the quotes behind any `ignored` sighting; then every line held back by a guard, every line excluded as too new, and the inert lines waiting on a second window; then `To relay`, one unchecked checklist line per public branch with its receive command, or the words `nothing to relay`; then a funnel counting sessions in the window, sessions scored, sightings returned, sightings dropped for a missing quote, lines measured, and lines proposed for removal. Label the pull request `relay` when the checklist is non-empty.

14. With `--report-only`, do everything except steps 11 through 13, print what would be retired and what `retired.md` would record, and write nothing to either memory repository. Still commit the replacement `scoreboard.tsv` in each traces checkout it touched.

## Why a removal is gated this way

The scoring signal is weak in one direction only. A line that drew a `steered` sighting demonstrably worked, but a line that drew nothing may have been silently obeyed, may govern a domain the window happens not to contain, or may be read by a kind of session the window excludes. So disuse never removes a line on its own: it removes a line that is also old enough for the whole window to have tested it, is not one of the two shapes whose disuse is known to be uninformative, and — when the only evidence is that it was ignored — has already survived one window and one chance at reinforcement.

The measured example this was built against: a 178-session window over both indexes left six of roughly ninety lines with no sighting of any kind, and five of the six were a topic pointer or a hygiene rule. Expect the eligible set to be small and the report to be most of the value.

## Self-exclusion

Every prompt sent by this skill must begin with `<!-- memory-backpass:self -->`, as [score-prompt.md](score-prompt.md) does. `allod trace distill` drops a session whose first user message starts with that line, so neither memory pass scores its own model calls. Preserve the marker when retrying or adding diagnostic text to a prompt.

## Failures

Stop with a one-line report when a traces checkout is missing, a required model is unreachable, or a credential is refused. Never fall back to a model the roster does not name for this skill, and never leave a model switch out of the run report. A run never pushes to a memory repository's default branch and never merges.
