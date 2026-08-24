# Shell Gotchas

Each turns a check into one that always passes. A guard that cannot be shown to fail on sabotaged input does not count (`architecture.md` principle 11).

## Human-run checks must not change the interactive shell's failure policy

Never ask a human to paste a multi-command block that enables `set -e`, `set -u`, or `pipefail`, or that calls `exit`. A failed check can terminate their interactive shell and close the terminal tab. Give one command at a time with the expected result. If checks genuinely need shared state or automatic stopping, run them in an explicit child shell or script so failure exits only that child, never the caller's shell.

## `set -e` exempts inverted and non-final commands

`! cmd` never aborts under `set -e`, so a `! rg <forbidden-token>` scrub assertion is a silent no-op. Use `if rg <forbidden-token> .; then exit 1; fi`.

In `a && b && c` only the final command's failure aborts, so an assertion chain ending in `echo` can never fail. One assertion per line.

## `jq -r` prints `null` for a missing path

A missing path yields the four characters `null` with exit 0, defeating `[ -n "$value" ]`. Append `// empty`.

## `git worktree prune` exits 0 when it cannot delete

A prune that fails to remove an admin entry — an unwritable `.git/worktrees/<name>`, say — prints `error: failed to delete ...` to stderr and still exits 0, leaving the worktree listed. Assert the post-condition (`git worktree list --porcelain` no longer names it), never the exit status.

## A probe that cannot distinguish absence from denial

`ls <dangling-symlink>` prints the link name and exits 0 — no `stat` of the target is needed for a bare argument. An access probe ending in `ls <link>` therefore passes whether or not the target exists. Force resolution with a trailing slash: `ls <link>/`.

`rm -f <path>` exits 0 when the path is absent, including when a parent directory is, so it cannot show that a deletion was refused. Use `unlink`.

Both matter most in isolation fixtures, where "the command failed" is the evidence: a probe that succeeds against nothing reads as a successful attack, and one that succeeds vacuously reads as a closed boundary.

## `ssh-keyscan` writes its banner to stdout

The `# <host>:<port> SSH-2.0-<version>` line lands on stdout beside the keys, so `2>/dev/null` does not remove it. A field-extracting comparison then holds two lines and never matches the registry, and a "the host offered exactly one key" count reads the banner as a key, so a host offering a second one passes. Drop comments first: `ssh-keyscan ... | grep -v '^#'`.

## Double-escaped metacharacters in single quotes

Single quotes do no backslash processing, so `'\\+'` — meant as a literal `+` — reaches the engine as an escaped backslash plus the `+` quantifier: one or more literal backslashes. Escape once: `'\+'`. Applies to `rg` and `grep -E`; in BRE a bare `+` is already literal.
