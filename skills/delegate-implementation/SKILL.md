---
name: delegate-implementation
description: Split an implementation arc across agents: one overseer on the most capable model available that briefs, waits on callbacks instead of polling, verifies and reports, with cheaper workers doing the scoped work in their own worktrees and reviewers from another model family reading the result cold. Covers what the overseer does and never does, how to wait without spending tokens, how to keep the overseer's context small, how to scope a worker so its output is usable, and what the overseer must run itself before a PR body claims it. Names no harness, CLI or provider. Use when one session is about to implement more than one issue, or when a change needs a second opinion from a model that did not write it.
---

# delegate-implementation

One session implementing five issues on the most capable model available is
the expensive way to get five mediocre PRs. The shape that works is one
overseer, several workers, and reviewers who did not write the code. This
skill is the method and names no harness, CLI, provider or account: which
models exist, what they cost and what each is good for is the deployment's
knowledge, and a deployment says so in a private skill named `agent-roster`.
If your memory index lists one, read it before choosing a model. `list-models`
shows what the machine you are on can reach today and verifies that a named
model answers.

The shape is Kun Chen's firstmate (github.com/kunchenguid/firstmate), reduced
to what this repository's workflows need.

## The overseer

The overseer is the session the human talks to, and it runs on the smartest
model available, because everything it does is judgment: reading the whole
arc, deciding the order, writing each worker's brief, verifying what comes
back, and writing the PR body the human will merge on. It does not implement.
It never edits a project file; a worker does that, in a worktree the overseer
created for it. When the overseer catches itself opening an editor on the
work, it stops and writes a brief instead.

Three kinds of decision, written down before the first dispatch, so the
overseer never invents policy mid-arc:

- **decide alone:** ordering, worker briefs, which findings are real, when a
  worker is stuck;
- **ask the human:** anything that changes scope, touches a shared interface
  the issue did not name, or would merge, publish or discard work;
- **never:** merge, push past a rail, delete unlanded work, or report a
  validation it did not see run.

## Waiting without spending

Supervision costs nothing when the overseer is asleep and everything when it
polls. A turn spent asking "done yet?" is tokens spent on no information.

- **Wait on callbacks.** Dispatch a worker and end the turn; the harness wakes
  the overseer when the worker returns. A long command runs in the background
  and re-invokes the overseer when it exits. Where the harness offers no
  callback for a condition, run one watcher that sleeps until the condition
  holds and then exits, so it wakes the overseer exactly once.
- **One live wait per job.** A second watcher on the same job is a second
  wake for the same event.
- **Empty checks are not progress.** Elapsed time, an unchanged status and a
  no-change poll are not worth a turn, a message or a line in a report.
- **Never end a turn blind.** With work under way, a wait is armed before the
  turn ends; a turn that ends with workers running and nothing to wake the
  overseer strands them. A scheduled wake-up is the fallback for external
  state no callback covers, at the cadence that state actually changes, never
  as a heartbeat.
- **State lives on disk.** The arc's plan, each brief, and each report are
  files, so a restarted overseer resumes from them instead of from memory of
  a lost transcript.

## Keeping the overseer small

The overseer's context is the scarce resource, and a worker's transcript is
the fastest way to fill it.

- A worker's tool output stays in the worker. The overseer reads the report,
  not the transcript, and opens the diff only to verify a claim.
- Reports are short and fixed in shape: what changed, what was run, its
  actual output, what was not run, and what needs a decision.
- Independent dispatches go out in one turn, not one per turn; the callbacks
  arrive as they arrive.
- The overseer re-reads nothing it already has. A brief written to disk is
  cited by path, not pasted back.

## Scoping a worker

A worker only does what its brief lets it do, and it runs on the cheaper tier
your harness offers for delegated work; the search tier, cheaper again, is for
lookups, because finding where a symbol lives is not the same job as changing
it. Every brief carries:

- the issue reference and its full body, not a paraphrase;
- the worktree path, created by the overseer with `allod change begin -d`, so
  two workers never share a checkout;
- what done means, as commands: the tests, the checks and the evaluation that
  must be green, with the instruction to report their actual output;
- the boundaries: no push, no attribution trailers, nothing private in a
  public repo, and the memory tree to read first;
- what is out of scope, named, so the worker does not wander into the next
  issue.

Delegation does not pay for one small issue, or for a change whose design is
the work. If the brief would take longer to write than the change, make the
change.

## Reviewing across families

Reviewers are cheap next to the defect they catch: a second reviewer on code
that had already passed one review has found real defects the first missed,
including a symlink escape that would have published a system file through a
public web root. Use a model that did not write the change, from another
family where the roster offers one, and run it after the final fix so it
reads the code that will land. Two reviewers per artifact, each reading the
diff cold, each asked for findings with a file and line, the claimed failure
and the input that triggers it.

Verify every finding against the source before acting on it. Reviewers
confidently invent bugs in code they did not run; a finding the overseer
cannot reproduce or trace is discarded, not fixed. A review that returns
nothing is checked for emptiness before it is read as approval, and a
scripted call to any model is gated on its content, never on its exit status:
a refused model, an unreachable provider and a hung call all look the same to
a script that checks only that the command returned.

## Failures

- A missing tool, a refused credential or an unsupported model is a blocker
  to report, never a reason to switch model or backend silently.
- A worker whose report claims a validation it did not show ran is sent back
  for the output, not trusted.
- A stuck worker is inspected, then given a narrower brief or stopped; its
  worktree is never discarded while it holds unlanded commits.
- Failures are reported plainly, first, with the evidence. A red check is a
  red check in the report, not a footnote.

## What the overseer runs itself

A worker's report says what it ran. Before the PR body claims it, the overseer
runs it again, or reads the worker's captured output line by line. A
validation claim in a PR body that was not run, or was run more narrowly than
the body states, is a defect in the change. The same goes for a reviewer's
finding: the overseer, not the reviewer, decides it is real.

## Precise limits

- This skill names no provider, account, CLI or harness. Model tiers, callback
  mechanisms and watchers are whatever the harness offers; a deployment's
  `agent-roster` says which exist and what they cost.
- Where the harness has no subagents, a worker is a separate session with the
  same brief, and the callback is a watcher on the artifact it produces.
- The overseer's model choice is the one place this skill is opinionated:
  cheaper workers are a saving, a cheaper overseer is a false one.
