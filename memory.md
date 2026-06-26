# Allod Memory

Allod is a self-sovereign NixOS VM stack for agentic coding and privacy tasks.

## Topic Files
- `allod.md` - Allod overview, repository inventory, and Forge CLI notes
- `git-workflow.md` - branching strategy, Forge CLI usage, issue and PR body formatting
- `dev-plans.md` - development plan requirements and review process
- `security-practices.md` - token handling and authentication safety
- `vm-tooling.md` - VM package policy
- `vm-provisioning.md` - provisioning stack, source-of-truth pointers, and gotchas
- `nix.md` - NixOS gotchas
- `age.md` - age and agenix workflows

## Memory File Hygiene
- `memory.md` is the only root memory file.
- Add durable memory to the listed topic file that owns it, or update this index when adding a new topic file.

## PR Workflow
- When the user suggests a change to an open PR, comment on the PR recording the request before implementing it.
- Link every implementation PR to the tracking issue. For multi-repo work, use `Refs` on earlier PRs and `Closes` only on the final integration PR.
- When manually closing a PR, delete its remote branch with `forge pr close ... -d`.
- After opening a PR, run or request a read-only review pass, comment findings on the PR, then implement fixes in a follow-up commit.

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

## Subagent Usage
- Do not launch subagents and do the same work directly. Pick one owner for each task.
- Use direct reads when the important files are already known.
- Use subagents for broad exploration of unknown areas or independent review passes.
