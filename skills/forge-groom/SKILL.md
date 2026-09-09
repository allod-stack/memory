---
name: forge-groom
description: Triage every open issue and pull request in one or more allod repos against what has actually landed on master, close only what a merged Closes line proves settled, label the rest, and produce one short plain-text report for the owner. Use for the weekly timer run, or by hand when a tracker has drifted from reality.
---

# forge-groom

A groom answers one question per open item: what is true about it today that
nobody has written down? Most tracker bloat is items whose answer changed
weeks ago: a fix that landed without closing its issue, a PR whose recorded
blocker was fixed on master, a decision issue whose trigger fired. The groom
finds those mechanically and hands the owner the few real decisions left.

It runs with the forge token on the machine, so its authority is bounded by
rule, and on the timer also by a token scoped to issue write and repository
read (see the dev plan in `allod/strategy` `dev-plans/forge-groom.md`).

## What it may do

| Action | Allowed when |
| --- | --- |
| Close an issue as completed | A pull request in `merged` state has `Closes owner/repo#N` for it in its body. Quote the PR number and title in the close comment. Nothing else closes an issue. |
| Set a label from the groom set | Always, on issues and PRs, only these labels: `landed?`, `ready-to-merge`, `blocked`, `stale`, `decision`. Remove a groom label the run no longer supports. Touch no other label. |
| Write the report | Always. |

Never: close an issue as not planned or duplicate, close or merge a PR, edit
a title or body, comment on a thread beyond the one close note, push, fetch
into a checkout the owner uses, or create labels in a repo that lacks them
(report the missing label instead; a human creates it once).

A `Refs owner/repo#N` in a merged PR is not evidence the issue is done: memory
reserves `Refs` for the earlier PRs of a chain. Label the issue `landed?` and
list it under "Likely landed, confirm".

## Inputs

- Repos: every registry checkout under the workspace root (the same set
  `workspace_collect_repos` yields), or the list given on the command line.
- `forge` on PATH, authenticated (`forge auth status`).
- A scratch directory for clones. All git work happens in throwaway clones of
  `origin`; never operate on the owner's checkouts.
- Age thresholds: `stale` after 30 days without a comment or commit,
  `decision` surfaced after 30 days open.

## Steps per repo

1. `forge -R <repo> issue list` and `forge -R <repo> pr list`, then
   `forge -R <repo> pr list -s closed` for merged evidence (lists PRs in
   `closed` state with a `merged` marker). Read each open item with
   `forge issue view` or `forge pr view`; comments count.
2. Clone `origin` into scratch, fetch every open PR head, and for each PR test
   `git merge-tree --write-tree origin/master <head>`. Clean and its base is
   master: candidate for `ready-to-merge`. Conflicts, or its base is another
   open PR, or its last review comment lists findings without a later
   "addressed" reply: `blocked`, naming why in the report.
3. For each open issue, look for evidence on master, in this order, and stop
   at the first hit:
   - A merged PR whose body has `Closes owner/repo#N`: close as completed.
   - A merged PR whose body has `Refs owner/repo#N`, or a commit on
     `origin/master` whose message names `#N` or `owner/repo#N`: label
     `landed?`, report under "Likely landed, confirm" with the PR or commit.
   - The body carries a decision shape (numbered options, "decision
     criterion", "close as not planned when"), or the label `decision`, and
     the issue is older than the threshold: label `decision`, report it as one
     yes-or-no question the owner can answer by reply.
   - No activity past the threshold: label `stale`, count it.
   - Otherwise: still open, count it.
4. Recheck before any close: view the PR again and confirm it is merged, not
   merely closed. A closed-unmerged PR is not evidence.

## Report

One plain-text document per run, under about 80 lines, headed by a one-line
summary: `<date>: <n> closed, <m> ready to merge, <k> decisions, <s> stale`.
Then one section per heading below, repos as subheadings, one line per item
with its URL. Omit empty sections. No Markdown tables; the reader is a mail
client.

```
Closed with evidence
  allod/tools#172  fixed by PR #175 (merged 2026-09-08)

Ready to merge
  allod/tools#166..169  site credential chain; merge in order 166, 167, 168, 169

Decisions for you (reply yes/no)
  allod/tools#124  auto-rebase in change record: close as not planned? (no rejection reported since July)

Likely landed, confirm
  allod/tools#118  moved-checkout guard: PR #131 Refs it; sweep guard not found on master

Blocked
  allod/archetypes#24  conflicts with master in modules/secrets.nix

Still open: 23 (stale: 6)
```

Write it to the path in `GROOM_REPORT` when set, otherwise to stdout. Mailing
is the caller's job, not the skill's.

## Running it

By hand from any harness, in the workspace root, after reading this file:
"Run the forge-groom skill over allod/tools and print the report." Give the
harness the repo list and the scratch path.

Headless on the timer (the archetypes module owns the unit; shown for
reproduction by hand):

```bash
timeout 1h pi -p --no-session --no-extensions --no-prompt-templates \
  --no-context-files --no-approve \
  --model finite/deepseek-v4-flash-0731-thinking --thinking medium \
  < prompt.txt > "$GROOM_REPORT"
```

`prompt.txt` is this file followed by the repo list and the scratch path.
Wrap in `timeout`: pi never retries a stalled stream. The `--no-*` battery
keeps repository content from loading extensions or skills of its own.

## Guards that make a run trustworthy

- A run that cannot reach the forge, or whose token fails `forge auth
  status`, writes a one-line report saying so and exits non-zero. It never
  reports a tracker as clean because it could not read it.
- Every close in the report names the merged PR; every label change is
  listed. A run's actions are a subset of its report, never the other way
  round.
- Idempotent: a second run over an unchanged tracker performs no actions and
  reports the same counts.
- A weaker model reads this file as its whole contract. Keep the rules
  positive and enumerable; anything the skill does not allow is forbidden.
