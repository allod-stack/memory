---
name: verify-merge
summary: Prove a PR behaves through its feature map, then merge it cold
description: Verify a pull request by driving its behavior through the repository's feature map, post the evidence, and merge it under the risk ladder in git-workflow.md. Runs in a session that did not write the PR, with the verifier credential, and stops instead of merging whenever the score, the rung, the coverage or the base has moved under it. Use when a PR is ready to land in a repository where agents hold merge rights, or when the owner asks for a merge digest.
---

# verify-merge

A merge is the one act the stack lets no agent take by default, and this
skill is the exception: a cold session proves a pull request does what it
claims, writes the proof where a skeptic can read it, and only then presses
the button. Evidence, never assertion. When a step below cannot be completed
the answer is a comment saying so and no merge, never a merge with a caveat.

The ladder (`git-workflow.md`, Merge Rights) says which risk class an agent
may merge in which repository. The forge enforces three things: the author
account cannot merge at all, the merge call is pinned to the head commit it
verified, and the merge is read back from the base branch. Everything else in
this skill is a rule the verifier follows, and the proof comment is how the
owner sees whether it was followed.

## Inputs

- A pull request, by number and `owner/repo`.
- The verifier credential. `forge auth status` must print the verifier
  account, not the author account; the deployment's private memory says
  where that token lives and how to select it (`FORGE_TOKEN_FILE`). The
  author account prints its own name there, which is the signal to stop: this
  session is not the verifier.
- The repository's rung ledger: a forge issue, one per repository, named in
  the deployment's private memory, where the owner's comments set the rung
  per risk class. The latest owner comment wins. No ledger, or no comment for
  a class, means human gate for that class.
- A scratch clone of the repository's `origin`. Never verify in the owner's
  checkout or in another agent's worktree: `git clone` into scratch, fetch
  `refs/pull/<n>/head`, and check out the head commit by hash.
- The repository's `verification/features/README.md`, when it exists. It
  names the driver commands (launch, health check, drive, inspect, clean up),
  the baseline preconditions, and the proof standard. Without it the proof
  is the checks plus a rerun of the PR body's Validation commands; the map
  is what lets the verifier drive beyond what the author ran.

## Steps

1. **Confirm the seat.** `forge auth status` names the verifier account.
   `allod change list` shows no worktree of this session on the PR's branch.
   Read the PR body's `## Validation` section: it must name commands that
   were run and show their output. Missing, or claims with no output, is a
   stop: comment that the PR needs its author's validation, and hand it back.
2. **Score it yourself, first.** Read the diff before the body's score. Walk
   `risk.md`'s calibration questions and write the class down with the one
   fact that decided it. Then read the body's score; the higher of the two
   is the score. A diff that touches `verification/features/` is diffed on
   its own: any removed file, removed entry point, removed path or removed
   "what usually lies" item sets the floor at R2, because a narrowed map is
   how a verifier is made to pass. A diff that removes or weakens a check
   has the same floor.
3. **Look up the rung.** Read the ledger issue's latest owner comment for
   this class. Human gate: post the verification (steps 5 to 7 still run, so
   the owner gets the proof) and stop before step 8. Agent merge: continue.
   R2 adds a veto window: the merge happens no sooner than 24 hours after
   the proof comment, and the comment states the earliest time, unless the
   ledger comment waives the window for that repository.
4. **Pin the revision.** Record `head.sha`, `base.ref` and the base tip
   (`GET /repos/{repo}/branches/{base}`). The PR must be open, unmerged,
   `mergeable`, based on the default branch, and not stacked on another open
   PR. Check out the head hash in the scratch clone.
5. **Run the checks.** The checks the feature-map README names as baseline,
   or the repository README's own when there is no map, at the head commit,
   with output captured. A failed check is a finding, never rerun to green.
6. **Drive the map.** R0 has nothing to drive; the proof is the checks plus
   the diff read, and for a memory repository the byte-budget hook. With no
   map, rerun the PR body's Validation commands at the head and capture
   their output; they are the coverage table. With a map, for R1 and up:
   match every touched area to a feature file; an area with none is
   a coverage gap. Run the health check first, then every reachable entry
   point the file lists, and the success, cancel, error, empty and
   persistence paths the change can affect, each with its expected
   observable result. When the PR claims a behavior difference, drive the
   same entry points once on the base tip so the proof shows before and
   after. Capture evidence as command output tied to the head hash.
   A path that cannot be reached is named with what blocks it. Any gap in
   the change's coverage set is a stop: proof posted, no merge, PR handed to
   the owner. A map that no longer matches the product is a finding for the
   author, not something the verifier edits.
7. **Post the proof.** One comment, `forge pr comment <n> -F <file>`, in
   the shape under "The proof comment". Findings go in the same comment and
   back to the author; a new head means a new verification from step 4.
8. **Re-read the base, then merge.** If the base tip moved since step 4,
   build the merged tree (`git merge-tree --write-tree <base> <head>`), run
   the checks on it, and for R1 and up re-drive the affected feature files;
   a conflict is a stop. Then
   `forge pr merge <n> --rebase --match-head-commit <head.sha>`. The command
   refuses a moved head, surfaces the forge's refusal verbatim, and exits
   non-zero when the base does not contain the merge commit afterwards
   (allod/tools#246); any of those is reported as is, never retried.
9. **Record the landing.** A second comment:
   `Merged as <merge_commit_sha>. Revert: git revert <base-tip>..<merge_commit_sha>`
   where `<base-tip>` is the tip read in step 8. That line is what the owner
   runs on master; the verifier never pushes to a default branch.

## The proof comment

```
## Verification

Verifier: <account> on <hostname>, scratch clone <path>
Head: <sha>   Base: <branch> at <sha>
Score: R<n> — <the deciding fact>. PR body says R<m>.
Rung: R<n> on <owner/repo> is <agent merge | human gate> (<ledger issue>, owner comment <date>)
Checks: <command> — <pass | fail>, output below
Coverage:
| Feature file | Entry point and path | Expected | Observed | Evidence |
| ... |
Gaps: none | <path> — <what blocks it>
Decision: merge | stop — <reason>
```

Trimmed command output follows in fenced blocks. Every row names the
expected result before the observed one, so a reader can tell a pass from a
"it ran".

## Stops

Never merge when: the seat is the author account; the session wrote the PR;
the Validation section is missing or unexecuted; the walked score is above
the rung; the map or a check was narrowed and the rung is below R2; a
coverage path is a gap; a check failed; the base moved and the rerun did not
pass; the PR is stacked, closed, merged or not mergeable; the merge command
exits non-zero. Every stop is a comment naming the reason, so the owner never
has to guess why a PR sat.

Never: edit the feature map, the checks, the ledger or the rung; approve the
PR on the forge; push to a default branch; merge with `--merge` or
`--squash` unless the repository's README says so; merge on a human-gate
rung "because it is small".

## The digest

`skills/verify-merge/merge-digest [-d <days>] <owner/repo>...` lists every PR
merged in the window per repository: number, title, merge date, whether a
`## Verification` comment exists (an agent merge) and the revert line from
the landing comment. The owner runs it on demand and at the week review in
`git-workflow.md`; it needs only `forge` on PATH. Promotion and demotion are
the owner's comment on the ledger issue; this skill reads it and never
writes it.

## Precise limits

- This skill names no product, driver or harness. The repository's
  `verification/features/README.md` carries the commands; `testing.md`
  (Feature Map) says what that file must hold, and
  `skills/verify-merge/features-README.md` is the template.
- Which machine holds the verifier token, which ledger issue belongs to
  which repository, and which repositories are in probation are deployment
  facts in private memory, not here.
- The forge cannot tell the author's session from the verifier's within one
  machine. The seat check in step 1 is a rule; the account split is the wall.
