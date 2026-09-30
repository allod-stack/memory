# Shell Gotchas

Each turns a check into one that always passes. A guard that cannot be shown to fail on sabotaged input does not count (`architecture.md` principle 11).

## Human-run checks must not change the interactive shell's failure policy

Never ask a human to paste a multi-command block that enables `set -e`, `set -u`, or `pipefail`, or that calls `exit`. A failed check can terminate their interactive shell and close the terminal tab. Give one command at a time with the expected result. If checks genuinely need shared state or automatic stopping, run them in an explicit child shell or script so failure exits only that child, never the caller's shell.


## An interactive `read` inside a pasted block eats the next line

Script input and `read` input are the same stream, so a pasted block arrives line by line and `read` consumes the *following* pasted line as its value — silently under `-s`, where it looks like nothing happened at all. When the buffer ends before a newline it blocks instead, which the operator reads as a hang and kills. Give the `read` as its own command on an idle terminal, and have it report what it captured (`echo "captured ${#K} chars"`) so "did it read anything" is answered immediately.

The guard on that value must be an `if` block. `[ -n "$K" ] || { echo …; return; }` stops nothing: outside a function `return` fails with `can only 'return' from a function or sourced script` and execution continues into the next command. `exit` in the same position does stop, by closing the human's shell.

## `set -e` exempts inverted and non-final commands

`! cmd` never aborts under `set -e`, so a `! rg <forbidden-token>` scrub assertion is a silent no-op. Use `if rg <forbidden-token> .; then exit 1; fi`.

In `a && b && c` only the final command's failure aborts, so an assertion chain ending in `echo` can never fail. One assertion per line.

## `set -e` is suspended inside an `if` condition, and inside every function called from one

A test runner written `if run_the_test; then PASS; else FAIL; fi` enforces only the last command of the test function: bash turns `-e` off for the whole condition, including the function body, so a failed `[[ ... ]]` in the middle falls through. A suite of thirty tests with several assertions each can report all green while pinning almost nothing. Run the body in a subshell that sets `-e` itself and read the status as a plain statement: `set +e; ( set -e; "$@" ); status=$?; set -e`. The same suspension applies to the left side of `||` and `&&`, so `cmd || status=$?` has the same hole.

## `read -p` prints no prompt when stdin is not a terminal

Bash writes the prompt only when input comes from a terminal, so a test that pipes an answer into a script and asserts on the prompt text (`Paste new token:`) can neither pass nor fail for a reason. Assert on the prompt's effect instead: what the answer was used for, or that a second line fed on stdin never reached anything.

## `jq -r` prints `null` for a missing path

A missing path yields the four characters `null` with exit 0, defeating `[ -n "$value" ]`. Append `// empty`.

## `git worktree prune` exits 0 when it cannot delete

A prune that fails to remove an admin entry — an unwritable `.git/worktrees/<name>`, say — prints `error: failed to delete ...` to stderr and still exits 0, leaving the worktree listed. Assert the post-condition (`git worktree list --porcelain` no longer names it), never the exit status.

## A probe that cannot distinguish absence from denial

`ls <dangling-symlink>` prints the link name and exits 0 — no `stat` of the target is needed for a bare argument. An access probe ending in `ls <link>` therefore passes whether or not the target exists. Force resolution with a trailing slash: `ls <link>/`.

`rm -f <path>` exits 0 when the path is absent, including when a parent directory is, so it cannot show that a deletion was refused. Use `unlink`.

Both matter most in isolation fixtures, where "the command failed" is the evidence: a probe that succeeds against nothing reads as a successful attack, and one that succeeds vacuously reads as a closed boundary.

## `systemctl` exit codes that read as absence

`systemctl --user stop <unit>` exits 5 with `Unit ... not loaded` for a unit that never ran or was already collected, so a stop cannot double as the test for "was it running". `is-active --quiet <unit>` exits 0 for active, 3 for failed or activating, 4 for inactive or no such unit, and 1 when the user manager cannot be reached; reading every non-zero as "not running" reports a machine whose manager is unreachable as clean. `list-units <pattern>` exits 0 with no output for no match and 1 for an unreachable manager, so `|| true` on it turns a failed probe into an empty list. Measured on systemd 260.

## `websocat` in line mode hides an empty frame

A WebSocket text frame of length zero prints nothing in `websocat`'s line mode, and live-server's reload signal is exactly that frame, so a reload probe built on `websocat` alone reports that reload is broken. Read the bytes after the handshake instead: `od -c` shows `0x81 0x00`.

## `ssh-keyscan` writes its banner to stdout

The `# <host>:<port> SSH-2.0-<version>` line lands on stdout beside the keys, so `2>/dev/null` does not remove it. A field-extracting comparison then holds two lines and never matches the registry, and a "the host offered exactly one key" count reads the banner as a key, so a host offering a second one passes. Drop comments first: `ssh-keyscan ... | grep -v '^#'`.

## Double-escaped metacharacters in single quotes

Single quotes do no backslash processing, so `'\\+'` — meant as a literal `+` — reaches the engine as an escaped backslash plus the `+` quantifier: one or more literal backslashes. Escape once: `'\+'`. Applies to `rg` and `grep -E`; in BRE a bare `+` is already literal.

## `| head` kills the producer after its lines are taken

A script piped into `head -N` gets `SIGPIPE` on its next write after `head` exits and dies there, so a multi-step script whose progress lines are trimmed with `head` runs only its first steps, and every output file a later step would have written is missing without any error. Write to a file and `head` the file, or trim after the script has finished.

## `ssh` re-parses its command in a remote shell, so one argument can arrive as several

`ssh host cmd arg1 arg2` does not forward an argv. It joins the arguments with spaces into one string and the remote login shell parses that string, so any argument containing whitespace or shell metacharacters is re-split on the far side. A Go template argument is the usual way to meet this: `--format '{{json .Config.Env}}'` reaches the remote `docker` as `{{json` and `.Config.Env}}`, which fails with `template parsing error: template: :1: unclosed action`, while a space-free `--format '{{.Id}}'` survives and hides the problem until someone writes a template with a space in it. Quoting at the call site does not help, because the local quotes are consumed locally.

Pass each argument through `printf '%q '` and hand `ssh` the single resulting string; the remote shell then reconstructs exactly the argv the caller passed. `%q` also escapes commas and other harmless characters, which round-trip correctly — worth checking once against the real remote for the argument shapes a wrapper actually sends, because this is a property of the boundary and no amount of reading the local source reveals it.
