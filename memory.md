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
- `nix.md` - NixOS gotchas
- `age.md` - age and agenix workflows

## Memory File Hygiene
- `memory.md` is the only root memory file.
- Add durable memory to the listed topic file that owns it, or update this index when adding a new topic file.
- Record state, not a changelog. Memory = current state of the world + decisions/gotchas that constrain future work. Git and the forge already log every merge/close — never mirror them. Drop "PR #N did X" narration; keep only its durable residue (a new convention, gotcha, or current fact) or nothing.
- Retire landed work. In-flight pointers are fine while live; when the work an entry tracks goes terminal (merged/closed/done), the edit noting that outcome instead deletes the entry or compresses it to its one durable fact. On every memory edit, sweep the section you touch for already-landed entries.

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
- Host-side commands require a human at the terminal.
- Host change workflow: agent edits, commits, and pushes; human pulls and runs host commands manually.
- Host-only commands include agenix re-encryption, SSH key generation, bootstrap scripts, VM rebuilds, and host-side VM repair.
