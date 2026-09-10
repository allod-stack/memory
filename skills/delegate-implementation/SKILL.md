---
name: delegate-implementation
description: Split an implementation arc across agents so the model you are running on plans, integrates and reviews while cheaper ones do the scoped work, and a model of another family reviews the result through pi. Covers when delegation pays, how to scope a worker so its output is usable, how to run pi from a script without it silently producing nothing, the gates that catch empty or refused calls, and what the integrator must run itself before a PR body claims it. Use when one session is about to implement more than one issue, or when a change needs a second opinion from a model that did not write it.
---

# delegate-implementation

One session implementing five issues on the most capable model available is
the expensive way to get five mediocre PRs. The shape that works is one
integrator, several workers, and reviewers who did not write the code. This
skill is the method; it assumes only that `pi` is installed, because the
framework puts it on every dev machine. A deployment usually knows more about
which models it can reach and what they cost, and says so in a private skill
named `agent-roster`; if your memory index lists one, read it before choosing
a model. Without it, `list-models` shows what this machine can reach today.

## The shape

- **The integrator** is your own session. It reads the whole arc, decides the
  order, writes each worker's brief, reviews what comes back, runs the
  validation itself, and writes the PR bodies. It does not implement.
- **A worker** is one subagent per issue, on the cheaper tier your harness
  offers for subagents, working in its own worktree. It gets a brief and
  returns a branch plus a short report of what it ran.
- **A reviewer** is a model of a different family from the one that wrote the
  change, run after the final fix, reading the diff cold. Two reviewers per
  artifact; verify every finding against source before acting on it.

Delegation does not pay for one small issue, or for a change whose design is
the work. If you would spend longer writing the brief than making the change,
make the change.

## Scoping a worker

A worker only does what its brief lets it do. Every brief carries:

- the issue reference and its full body, not a paraphrase;
- the worktree path, created by the integrator with `allod change begin -d`,
  so two workers never share a checkout;
- what done means, as commands: the tests, the checks, the eval that must be
  green, and the instruction to report their actual output;
- the boundaries: no push, no attribution trailers, nothing private in a
  public repo, and the memory tree to read first;
- what is out of scope, named, so the worker does not wander into the next
  issue.

Give the worker the search tier, not the implementation tier, for lookups:
finding where a symbol lives is not the same job as changing it.

## Running pi from a script

`pi -p` prints its answer and exits only when it has a terminal. Without one
it writes nothing and never exits; a script that redirects its output to a
file gets an empty file and a process that sits there until killed. Give it a
pseudo-terminal:

```
script -qec "pi -p --provider <provider> --model <model> '<prompt>'" /dev/null
```

Add `--mode json` for a machine-readable event stream: the `message_end`
event carries `message.content[].text`, `message.stopReason`,
`message.model` and `message.usage`. Attach files with `@path` before the
prompt; the reviewer reads the diff that way rather than through a pasted
blob. Output through `script` carries carriage returns; strip them with
`tr -d '\r'` before parsing.

A refused model, an unreachable provider, and a hung call all look alike to a
script that checks only the exit status. Gate on the content: the answer's
byte count, or the `message_end` event's text in json mode. `list-models`
probes any model you name with exactly this shape and reports served or
refused, so run it before scripting a reviewer around a model.

## Reviewing across families

Reviewers are cheap next to the defect they catch: a second reviewer on code
that had already passed one review has found real defects the first missed,
including one that published a system file through a web root. Run them
after the final fix, not before it, so they read the code that will land.

Ask for findings, each with a file and line, the claimed failure, and the
input that triggers it. Verify every one against the source: reviewers
confidently invent bugs in code they did not run, and a finding you cannot
reproduce or trace is discarded, not fixed. A review that returns nothing is
checked for emptiness before it is read as approval.

## What the integrator runs itself

A worker's report says what it ran. Before the PR body claims it, the
integrator runs it again, or reads the worker's captured output line by line.
A validation claim in a PR body that was not run, or was run more narrowly
than the body states, is a defect in the change. The same goes for a
reviewer's finding: the integrator, not the reviewer, decides it is real.

## Precise limits

- This skill names no provider and no account beyond `pi`. Which models cost
  what, and which are good at what, is the deployment's knowledge; see its
  `agent-roster` skill if there is one.
- A subagent's model tier is a harness feature. Where the harness has none,
  the worker is a separate session with the same brief.
