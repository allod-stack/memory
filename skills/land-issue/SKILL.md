---
name: land-issue
summary: Drive an accepted issue to a merged PR, or stop with a name
description: Drive one accepted issue to a merged default branch, or stop with the stage that stopped. Runs preflight, implementation (through delegate-impl when its triggers match), pull request, the cold verifier, cleanup and a landing notification, posting one run-log comment per transition on the issue. Caps verification at one fix round and classifies every stop. Use when the owner says "land this issue" or names an accepted issue to finish.
---

# land-issue

Every stage from an accepted issue to a merged default branch already exists
as a skill or a command; this skill is the driver that connects them and the
discipline that decides what ends a run. It takes `land-issue
<owner/repo>#<n>` (or `land-issue --pick`) and runs the whole chain in one
session, writing one comment per transition on the tracking issue so the
issue is the notification channel, the audit trail, and the crash-recovery
log. It never merges: the cold verifier does that under `verify-merge`'s
ladder, or it stops and the owner does.

## The termination contract

A run ends in exactly one of two states: `merged` (the default branch carries
the change and the clones are reclaimed), or `stopped` (a comment naming the
stage, the reason, and a link to the evidence). Exiting silently, or leaving
the run with an unread verifier comment and no notification, is a defect of
the run. Every transition is a comment, so a fresh session reads the last one
and resumes instead of guessing.

One fix round per verification stop, never two. The cap is `risk.md`'s pass
budget; it converts an unattended loop into a notification, and a second
consecutive verification failure is design signal that belongs to the owner,
not another round. The cap is this one line, revisitable when the logged pass
rates justify it.

## Preflight and single-flight

1. `work-diff`, then `pull-all`. A cold review forbids the pull, but this run
   owns the checkout, so both run.
2. Read the issue in full, comments included: `forge issue view <n>`.
3. Read the last `land-issue` comment on the issue. `merged` or `stopped`
   means the run is already terminal; say so and stop. `PR #<n>` means a pull
   request is open; resume at Verify. `implementing` means the clone exists;
   resume at Implement with `allod change begin --resume <description>
   <repo>`.
4. Single-flight: `allod change list <repo>`. A row on `agent/land-<n>` in
   state `running` is another driver holding the branch — refuse to start,
   post `stopped: single-flight, ...`, and stop. Two drivers on one issue is
   the collision the clone gate exists to prevent. An `unsubmitted` or `open`
   row is this run's own clone; resume it.
5. Post the `started` comment and cut the clones: `path=$(allod change begin
   -d land-<n> <repo>)`. For an issue that spans repos, cut a clone in each
   and name every one together on a single `allod change run`.

## The need, before the first edit

State the need in one sentence, the smallest change that meets it, and what
each feature it adds saves the owner, as a comment on the issue
(`agent-behavior.md`, Scope Discipline). This is the owner's veto: an issue
is its author's proposal, and a smaller change that meets the need is raised
here, not in a pull request. `--pick` runs `pick-next-issue` in ask-mode and
ends the run at the question — a designed stop — because the need statement
stays the owner's.

## Implement, record, submit

- `delegate-impl`'s own triggers decide the shape: more than one repo, or R3
  or higher, means read that skill and work as its overseer — workers edit in
  their own clones under `allod change run`, and a reviewer from another
  model family reads each diff cold before the body claims validation. Below
  both triggers, implement directly in the clone.
- Run the repository's checks (or the feature-map README's baseline) at the
  head before pushing. A red check is a red check in the run log, never a
  footnote.
- Record and submit from inside the sandbox: `allod change run <clone> --
  allod change record ...` and `... -- allod change submit -t <title> -F
  <body-file>`. The body opens with the owner summary (`memory.md`, PR
  Workflow), at most ten plain-English lines: what changes for a running
  machine, what stays identical, what proves it, and what to do if it is
  wrong. Then `## Risk` and `## Validation`. Link the issue; for a multi-PR
  arc use `Refs` on earlier PRs and `Closes` only on the final one.
- Post the `PR #<n>` comment with the link, then move to Verify.

## Verify, and react once

Start the cold verifier through the packaged wrapper, never a hand-rolled
pattern:

```
verify-session <owner/repo>#<n>
```

Its verifier credential comes from `--token-file` or `FORGE_TOKEN_FILE`; the
deployment's private memory names the file. The wrapper refuses the author's
credential and any live clone of the branch, cuts its own scratch clone at
`refs/pull/<n>/head`, and runs the verifier headless. The verifier reads
memory, follows `verify-merge`, and merges under the rung or stops. Post the
`verifying` comment with the round and, once the verifier returns, its stop
class and the proof-comment link.

- `verify-session` not on PATH is a designed stop for this run: post
  `stopped: verify-leg, verify-session not installed (allod/archetypes#153),
  ...` and stop with the manual `verify-merge` pattern named as the fallback.
- Read the result from the forge, never from the transcript: `forge pr view
  <n>`. A merged pull request ends the run; a proof comment with findings and
  an open pull request is a fix round.
- The fix round goes back to the same clone, still unpushed if the pull
  request has not opened: apply the verifier's findings as one more commit,
  re-record, and start the verifier over on the new head (`git-workflow.md`,
  Landing a PR). A pull request whose branch is already on the forge only
  gains commits.
- A second consecutive verification stop ends the run: post `stopped:
  verification, second failure, <link>` and notify the owner. Do not open a
  third round.

## Finish

- On merge: `allod change cleanup <clone>` for every clone, then post the
  `merged` comment with the merge commit and the revert line
  (`git revert <base-tip>..<merge-commit>`) the owner runs on master.
- On a designed stop (human gate rung, R3+, multi-repo, or `--pick`): post
  `stopped: human-gate, <proof link>, your call`. The proof is already on the
  pull request; nothing is needed from the driver.
- On an end-and-notify stop (second verification failure, base moved twice,
  a check still red after the fix): post the `stopped` comment with the link
  and stop.

## The run log

One comment per transition, on the tracking issue, in this shape. Keep it to
one line of fact plus the link; the forge, not this comment, holds the
detail.

```
**land-issue: started** — <owner/repo>#<n>, description land-<n>, clone <path>

**land-issue: implementing** — <direct | delegate-impl, workers: ...>, checks <command> <result>

**land-issue: PR #<n>** — <url>, score R<n>, validation <commands>

**land-issue: verifying** — verify-session, round <1|2>, stop class <none|findings|human-gate|...>, proof <url>

**land-issue: merged** — <merge commit sha>, revert: git revert <base-tip>..<merge sha>

**land-issue: stopped: <stage>** — <reason>, <link>, your call
```

## Stops

Three buckets, copied from the existing skills, never invented per run:

- **Fix in place** — verifier findings: one round, described above.
- **End and notify** — a second verification failure, the base moved twice,
  or a check still red after the fix; and any refusal the wrapper surfaces
  (author seat, live clone, unreadable credential or clone list).
- **Designed stop** — a human-gate rung (R3 and up, or any multi-repo
  change, per `git-workflow.md`, Merge Rights), `--pick` awaiting approval,
  and the verify leg before `allod/archetypes#153` installs the wrapper. End
  with the proof link and `your call`.

## Precise limits

- The driver never merges, never pushes a default branch, and never edits a
  feature map or a check to make a verifier pass. Merging is
  `verify-merge`'s act in the verifier's own session.
- The driver is prose and forge comments only; it adds no product code. The
  verifier's wrapper is `allod/tools`#305's, and the rung ledger, the
  verifier token and the ledger issue are deployment facts in private memory.
- `delegate-impl` decides the implementation shape by its own triggers; this
  skill does not override them, and `land-issue` on a single small issue is
  one direct session.
