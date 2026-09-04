---
name: fleet-diff
description: Gate a merge against which machines it actually rebuilds by evaluating every machine in a composition-root flake twice (committed lock vs. proposed revisions) and asserting the expected blast radius. Use before merging work that could reach a real machine, when a change is claimed land-inert, or when an override needs squaring that it actually took effect.
---

# fleet-diff

`fleet-diff` answers one question before you pour a branch into a real machine:
**which machines would this change actually rebuild?** It evaluates every
machine in a composition-root flake twice — once against the committed
`flake.lock`, once with the proposed revisions substituted — compares the two
`config.system.build.toplevel.drvPath` values, and reports `unchanged` or
`CHANGES` per machine. An expectation flag then fails when reality differs from
what the change claimed.

```
fleet-diff [<deploy-checkout>] --override <input>=<rev> [...]
           [--expect <machine>,... | --expect-none]
```

`fleet-diff` lives in `allod/tools` at `flake/fleet-diff`. Dev boxes get it as
the `fleet-diff` command on `PATH` through the archetypes home composition. You
rarely write this tool; you call the installed command.

## When to use it

- **Before any merge that could reach a real machine.** The land-inert case is
  the common one: a refactor, doc change, or dependency bump should alter no
  machine, and `--expect-none` is the gate that proves it.
- **For an activation change.** Name the machines the change is supposed to
  convert with `--expect`; the gate fails unless *exactly* those change.
- **To square an override.** A mistyped override path makes Nix warn, evaluate
  the baseline anyway, and exit 0 — which would report every machine unchanged
  and pass `--expect-none` while proving nothing. `fleet-diff` refuses an
  override path that names no input in `flake.lock`.

Do not use it for a security proof: it only compares drvPaths. It cannot see an
additional defect inside a machine already expected to change, external state,
or activation-only behaviour.

## How to run it

Run from inside the composition-root checkout (the default) or name one
explicitly. An override takes a **git revision**; which repository that
revision belongs to comes from the checkout's own `flake.lock`.

```bash
# Gate a land-inert merge: fail if any machine changes
cd ~/work/allod/deploy
fleet-diff --override archetypes/vm=1a2b3c4 --expect-none

# Gate an activation change: fail unless exactly these machines change
fleet-diff --override inventory=1a2b3c4 --expect allod-dev,nexus

# A source the lock cannot name: pass a whole flake URL in the same place,
# quoted — an unquoted & backgrounds the command
fleet-diff --override 'vm=git+https://forge.anarch.diy/allod/vm.git?rev=<40-char-rev>'
```

Key rules:

- **Abbreviate to 7 characters or more** and the revision is expanded against
  the remote's refs, so it has to be pushed and be the tip of a branch, tag, or
  pull request. Any other commit takes its whole 40-character revision, which
  Nix fetches directly without being told which ref carries it.
- **Transitive inputs override by path** (`archetypes/vm`), direct ones by name
  (`vm`). Get this wrong and `fleet-diff` tells you: a path resolving to nothing
  is a usage error, and a transitive input named without its path is rejected
  with a pointer at the path form.
- **Exit codes are distinct** so the tool composes as a preflight: `0` matches
  (or no expectation declared), `1` usage or precondition error, `2` expectation
  mismatch, `3` evaluation failure.
- The checkout needs both `flake.nix` and a **committed** `flake.lock`. An
  uncommitted one warns, because the baseline is then the working tree rather
  than the committed lock. Nothing is written: both evaluations pass
  `--no-write-lock-file`.
- Budget minutes, not seconds. Evaluation is per machine and sequential, so cost
  scales with fleet size; a nine-machine fleet takes around five minutes.

## Reading the output

The verdict goes to **stdout**; Nix's own diagnostics go to **stderr**. Keep
stderr — the "not writing modified lock file" block names each overridden input
with its old and new revision, and that is your receipt that the override took
effect. `fleet-diff … 2>/dev/null` gives a bare report if you need one.

Scope is the fleet the named checkout composes, and the tool says as much in its
own output. `allod/deploy` is the framework's example composition root; a
deployment's fork composes different machines, so a green run over one proves
nothing about the other. Run it in the composition root you actually intend to
merge into.

## Interpreting a mismatch

`--expect` and `--expect-none` each fail in **both** directions, and the two
sets are named separately:

- `changed, not expected:` — a machine changed that you said should not. Ask
  what about the branch reaches it. This is the common, dangerous case.
- `expected, did not change:` — a machine you expected to convert did not. Your
  override may not have taken effect, or the change is inert on it.

When a machine you thought you were touching shows `unchanged`, re-check the
override: a resolved-but-wrong one — pointing at the revision already locked,
say — gives no warning and is a silent no-op. Compare the "not writing modified
lock file" receipt on stderr against the revision you actually meant.

A machine can also change for a reason that has nothing to do with your edit.
Where a module interpolates a whole flake input rather than one file out of it —
`"${someSource}/dir/file"` — that input's source tree hash becomes a build
input, so *every* commit to that repository moves *every* machine composing the
module, whatever the commit says. `allod/archetypes` issue 21 tracks two
measured instances. Before reading `CHANGES` as evidence about your content,
establish whether the machine would have moved anyway.

## When Nix refuses the revision

`Cannot find Git revision <rev> in ref 'refs/heads/master'`, for a revision
`git ls-remote` plainly shows on master, is a stale Nix fetcher cache and not a
missing commit. Clear it with `nix flake metadata <url> --refresh`, then run
again. A forge that 503s mid-fetch is a separate and unrelated failure, showing
up as `RPC failed; HTTP 503` or a `could not get HEAD ref … using expired cached
ref` warning; retry rather than diagnose.

## Precise limits

- Per-machine sequential evaluation is deliberate, to dodge the memory peak a
  whole-composition-root `nix flake check` hits. It is not a speed bug.
- Do not normalise or massage drvPaths; the comparison is exact, and that
  exactness is the trustworthy part of the tool.
