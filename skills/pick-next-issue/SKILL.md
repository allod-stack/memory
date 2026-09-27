---
name: pick-next-issue
description: Sweep every open issue across the allod repos on the forge, verify the candidates against master, pick the one worth implementing next, explain it and the choice in plain English, and ask the owner before starting. Use when the owner says "what's next", "pick something", or "what else you got", or when a session has landed its assigned work and has capacity left.
---

# pick-next-issue

The owner asks "what's next?" and wants one answer they can say yes or no to
in under a minute, not a tour of the tracker. This skill is the sweep, the
shortlist, the check that each candidate is still real, the choice, and the
plain-English case for it. It ends with a question and stops. Nothing is
implemented until the owner answers.

## Sweep

- List every open issue in every registry repo: `forge -R allod/<repo> issue
  list` for each `allod/*` checkout under the workspace root. Skip nothing on
  the first pass; a repo with one issue may hold the pick.
- List open PRs the same way. An issue with an open PR is implemented and
  waiting on review, not available; the tracker cannot show that link
  (`allod/memory` issue #60), so read PR bodies for `Closes`/`Refs` lines.
- Read labels as the groom left them (`forge-groom`): drop `landed?`,
  `decision` and `blocked` from the shortlist. Keep `stale`: it means no
  activity, not low value, and the pick often carries it because its blocker
  was fixed months ago and nobody came back.

## Shortlist, then verify

Read each shortlisted issue in full with `forge issue view`, comments
included. Then check it against master before ranking it, because a tracker
lies in three ways:

- **Already done.** A merged PR fixed it without a `Closes` line. `git log
  --grep=<N>` on master and a look at the code the issue names.
- **Receipts rotted.** Line numbers and file names in an issue written before
  a port or a rename point at code that no longer exists. Re-locate every
  receipt; the defect usually survived the port (a Bash fallback ported
  verbatim into Go is the standing example) but the fix goes elsewhere.
- **Blocker gone.** An issue that waited on another one may be unblocked now.
  Read the dependency's state, not the label.

Drop anything the check shows is done. Note what you re-located; it goes in
the explanation.

## Rank

Value is what the change does for the owner's day, weighed against what it
costs to land. In order:

1. **It bites the workflow the owner actually runs.** The commands they type
   most (`allod change`, `pull-all`, provisioning, rotation) outrank a tool
   they may not use. When unsure whether a tool is used, that doubt is part of
   the ask, and a runner-up rides along.
2. **Unblocked and agent-doable.** No host-only step, no human-only
   validation, not waiting on a decision issue, not a design question wearing
   an implementation title.
3. **Done is executable.** A test, a check, a fixture, or a boot that settles
   it (`testing.md`). A spec proven by inspection is not a pick.
4. **Lowest residual risk for the value** (`risk.md`). A one-function change
   that removes a silent fail-open beats a port.
5. **Fits the session.** Same repo, same harness, same test suite as work
   already landed costs nothing to ramp.

Big arcs (multi-repo ports, CI runners, architecture changes) are planned
work, not picks. Bug reports are picks when the reproduction is in the body.

## Explain, then ask

Write for the owner cold, in the `writing.md` shape: plain first, receipts
last, no project jargon at first use.

1. **The pick**, one line: repo, number, title.
2. **What the issue says**, in plain English. What goes wrong, for whom, and
   what the fix makes true. An analogy is fine if it is exact.
3. **Why this one**, three or four bullets against the ranking above,
   including anything the verification step changed ("labeled stale because
   it waited on X; X landed; nobody read it since").
4. **What you passed over**, one line each for the two or three closest
   runners-up and why they lost.
5. **The question.** One line the owner can answer with a word: implement
   it? Offer the runner-up as the alternative when the pick's value depends on
   whether the owner uses the tool.

Then stop. Do not cut a worktree, do not start reading code for the fix, do
not draft a plan. A "no" or "what else" restarts at the shortlist with the
rejected pick and its reason recorded; the reason is a ranking signal ("I
never use that tool" demotes everything in that tool). A "yes" for one issue
is ordinary issue work; a "yes" for more than one goes through
`delegate-implementation`.

## What this skill is not

It does not close, label, or comment on anything; `forge-groom` owns the
tracker's state. It does not file issues. It picks one thing and asks.
