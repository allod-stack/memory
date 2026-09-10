# Agent Behavior

Check for a project tool before falling back to raw git, ssh, or shell: `allod change`, `allod patch`, `forge`, `pull-all`, `work-diff`, `flake-status`, with `allod --help` and `allod/tools/docs/` listing the rest. Handing the human a manual command sequence for something a tool already does is a defect.

## Scope Discipline

For issue work, identify the smallest repo-scoped artifact that advances the request, and make that change before exploring broader end-to-end fixes.

When an issue names multiple possible paths, separate them into immediate change, validation, manual test, and follow-up. Do not choose the most ambitious path by default.

Do not run expensive builds, package rebuilds, VM rebuilds, broad searches, local installs, or environment mutations to validate an alternate path unless the user asked for that path or the repo change cannot be selected without it.

For package/version tasks, prefer this order: bump the existing lock or package reference; do lightweight metadata checks if needed; add overrides only after the simple path is insufficient and accepted; build only when required by acceptance tests or explicitly requested.

Treat a user interruption as a hard scope signal: stop the current path, avoid adjacent exploration, and resume from the narrowest reading of the latest instruction.

Read user-provided paths and names literally. A leading dot (e.g. `allod/.profile`) is a real repo — the Forgejo org-profile repo — not a typo for a similarly named one; verify the exact path before acting on a look-alike.

Do not assume every machine is running. VMs are disposable and started on demand; unreachable is normal.

Several agents may run in one VM — supported, not accidental. A dirty tree or unexpected branch may be another agent's; leave work in progress alone and pick something else.


## Handing a Command to a Human

Anything a human copies out of a terminal can be mangled in transit, and the mangling is silent.

- Every relayed line is one complete command. A backslash continuation becomes two commands when the copy drops or relocates the backslash; the signature is a flag reported as a command (`bash: -R: command not found`) or an argument that simply vanished.
- Never inline a long opaque token. A wrapping copy inserts whitespace mid-token and the resulting error names the value rather than the wrap. Derive it from a file or a registry so the relayed line stays short.
- Never assume the working directory. Bake the `cd` into the command instead of describing it in prose above the block; the reader is not standing where you left them.
- Interactive input is its own command on an idle terminal, never a line inside a pasted block (`shell.md`).
- If a command is long, or the human is working in a captured session, write a readable script and use a hash-checked transfer: they inspect it, verify its hash after transfer, then execute the file. Never relay content the human cannot read, and never ask them to run an encoded blob.

When a command fails, the error signature names the mechanism. Re-derive the line from its source rather than hand-repairing mangled text.

## Crossing the Public/Private Boundary

An agent holding private material cannot write public org repos; pushes are refused server-side. That is the boundary working — do not look for another transport or open the change from a fork.

Hand off a public code change as a real commit on an `agent/<description>` branch in the public repo's own checkout, left unpushed with a clean worktree. Give the human one command — `allod patch receive <vm>:<source-repo> <dest-repo>`, where `<vm>` is your hostname — never a sequence of git commands. Do not pass `--push`: the relay applies the commits, and pushing them is the human's own step, so the change reaches a remote only after they have looked at it. Never embed the change as a patch in a notes or plan document; only sanitized issue and plan prose belongs there. Prepare it in the public checkout: a fork drifts and the diff may not apply.
