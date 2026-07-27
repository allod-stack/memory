# Allod Memory

Allod is a self-sovereign NixOS VM stack for agentic coding and privacy tasks.

## Topic Files
- `allod.md` - Allod overview, repository inventory, and Forge CLI notes
- `architecture.md` - core architecture principles; read before any architecture or design decision of consequence
- `git-workflow.md` - branching strategy, Forge CLI usage, issue and PR body formatting
- `issue-writing.md` - implementation issue scope and structure
- `dev-plans.md` - development plan requirements and review process
- `security-practices.md` - token handling and authentication safety
- `agent-behavior.md` - scope discipline and interruption handling
- `vm-tooling.md` - VM package policy
- `vm-provisioning.md` - provisioning stack, source-of-truth pointers, and gotchas
- `agent-cli-updates.md` - versioning and bumping the dev-VM coding-agent CLIs
- `nix.md` - NixOS gotchas
- `shell.md` - bash and jq gotchas that make assertions silently pass
- `age.md` - age and agenix workflows

## Memory File Hygiene
- `memory.md` is the only root memory file.
- Add durable memory to the listed topic file that owns it, or update this index when adding a new topic file.
- Keep entries maximally concise — maximum information, minimum tokens. State the current capability, limit, or fact plainly. Cut historical narration (dates, "verified on X", "the old note was wrong", issue and PR numbers that only recount events) and hedging prose. Keep a reference only when a future reader needs it to act.
- Record state, not a changelog. Memory = current state of the world + decisions/gotchas that constrain future work. Git and the forge already log every merge/close — never mirror them. Drop "PR #N did X" narration; keep only its durable residue (a new convention, gotcha, or current fact) or nothing.
- Retire landed work. In-flight pointers are fine while live; when the work an entry tracks goes terminal (merged/closed/done), the edit noting that outcome instead deletes the entry or compresses it to its one durable fact. On every memory edit, sweep the section you touch for already-landed entries.

## Public vs Private Memory

This repo is the public memory; a deployment keeps a separate private one. Private-access agents read both, so a public fact reaches every agent and a private one reaches only some. File by what the fact is about, not by which repo the task is in.

Public: conventions and process; public tool and hook behavior, including bugs and workarounds; framework facts about public repos; Nix, shell, and NixOS gotchas not tied to one machine.

Private: absolute paths, hostnames, IPs, usernames, account and key names, secret locations; private-fork specifics and token capabilities; human preferences and private-project notes.

Re-verify a fact before relocating it. An agent auditing only public memory cannot see the private repo, so a claim retired in one can survive in the other and be re-imported later.

A mixed fact splits — publish the mechanism, keep the specifics private, and have the private note lean on the public one. Component and repo names are fine in public, identity material never is (`architecture.md` principle 5). Publishing is irreversible (principle 10), so file the genuinely unclear case privately.

## PR Workflow
- When the user suggests a change to an open PR, comment on the PR recording the request before implementing it.
- Link every implementation PR to the tracking issue. For multi-repo work, use `Refs` on earlier PRs and `Closes` only on the final integration PR.
- When manually closing a PR, delete its remote branch with `forge pr close ... -d`.
- After opening a PR, run or request a read-only review pass, comment findings on the PR, then implement fixes in a follow-up commit.
- PR bodies should expose residual risk and validation signal when useful for human triage. Do not block PR creation solely over missing headings. Do not post no-findings update-check comments.

## Git Workflow
- Before starting work, run `work-diff` and then `pull-all`.
- Use `allod change` for change work: `begin`, edit, `record`, `submit`.
- Protected repos: `path=$(allod change begin -d <desc> <repo>)`; edit in `$path`; run `allod change record`; run `allod change submit`.
- Unprotected repo plus PR requested: create or switch to `agent/<desc>` from `master`, then use `allod change record` and `allod change submit`.
- PRs and issues use the `forge` CLI for Forgejo.

## Execution Architecture
- Agents run inside dev VMs.
- An agent's VM name is its hostname (`hostname`); use it for self-references like the `allod patch` source host.
- Host-side commands require a human at the terminal.
- Host change workflow: agent edits, commits, and pushes; human pulls and runs host commands manually.
- Host-only commands include agenix re-encryption, SSH key generation, bootstrap scripts, VM rebuilds, and host-side VM repair.
