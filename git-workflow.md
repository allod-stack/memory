# Git Workflow

- Before starting work: `work-diff` -> `pull-all`.
- Before treating a requested Allod file as missing or stale, ensure its repo is
  current. If `pull-all` fails there, inspect the repo: with a clean worktree on
  a gone upstream, switch to the default branch and fast-forward pull; otherwise
  read the file from `origin/HEAD` or state the freshness problem.
- Use `allod change` for change work instead of manual git and Forge mutation steps:
  - Start branch work with `path=$(allod change begin -d <short-description> <repo-path>)`, then work in `$path`.
  - Continue a branch the forge already has — a PR's fix round in a new session, or one whose clone was cleaned up — with `path=$(allod change begin --resume <description> <repo-path>)`: the same clone under `~/changes`, tracking `origin/agent/<description>`, so `record` and `submit` carry on. It refuses a branch missing from origin or already claimed locally, and cannot take `-d`.
  - Run a delegated worker, or anything else that should see nothing but the change, with `allod change run <clone-path>... [--memory <size>] -- <command...>`. Several clones is the cross-repo case: each is writable inside, and no other repository exists there.
  - Commit and push with `allod change run <clone-path> -- allod change record -m <message> [-f <file>...]`. A multi-line message comes from a file with `-M <file>`, or from stdin with `-M -`; `record` has no `-F`, which is `submit`'s flag for the PR body. A ledgered clone refuses `record` run directly, outside `run` (exit 11); a worker already inside its own sandbox calls `record` directly instead, since it is already there.
  - Open PRs the same way, with `allod change run <clone-path> -- allod change submit -t <title> -F <body-file>`.
  - Find reclaimable clones with `allod change list [<repo-path>]`.
  - Remove merged clones with `allod change cleanup <clone-path>`.
- `-d` is the isolation switch: it creates a self-contained clone under `~/changes` and an `agent/<description>` branch for every repo, protected or not. Without `-d`, `begin` prints the shared checkout path and creates nothing — the in-place flow for a repo's default branch, which protected repos refuse.
- `allod patch receive <vm>:<repo-id> <repo-id>` resolves a registry-id source to the repo's shared checkout (`~/work/<checkout>`) on the source machine, never to an `allod change` clone under `~/changes` — a change made in a clone must name the source by path (`<vm>:~/changes/<clone>`), or the relay resolves to the shared checkout, finds no unpushed commits there, and fails with an empty export range (exit 11).
- Before handing over a relay line for a worktree branch, fetch the target repo's `origin` and rebase the worktree onto it: the branch's base was recorded when the clone was made, and once master moves the receiver fails with `base <ref> is not an ancestor of HEAD` or lands the commit on a base its tree no longer has.
- `allod patch fetch --base <ref>` resolves the ref on the source machine and exits with `base <ref> is not an ancestor of HEAD` once the source checkout's master has moved past the branch point; name the branch point commit instead. Into a machine that cannot SSH out (a dev VM), the relay is `fetch --output <dir>` on nexus, `scp -r` of that artifact directory to the VM, and `allod patch apply <dir> --repo <clone>` there. Applying onto a clone cut from current master is the rebase: `git am --3way` lands the commit on the new base, and the conflict markers it leaves are inside heredocs that `bash -n` never sees, so grep for them before amending.
- `allod change submit` opens the PR but pushes nothing; only `record` pushes. A commit made another way — a relayed patch, an amend after a rebase — needs `git push -u origin <branch>` first, or submit fails with a bare `POST /repos/<owner>/<repo>/pulls failed: HTTP 404`.
- `protected-branches` governs only which branch is protected, never whether a change is isolated.
- Branch work always happens in its own clone, never by switching branches in a shared checkout — see Clones and Concurrent Agents.
- No external remote pushes unless the remote is explicitly allowed locally.
- PR branches: add commits, do not force-push.
- Do not stack a PR on another open PR. PRs land by rebase-merge, so master carries new hashes, and the forge refuses a force-push on `agent/*` (`force-push is not permitted ... Add commits instead`): the stacked branch conflicts the moment its base merges, and the local rebase that would fix it cannot be pushed. Wait for the base to merge, or cut the dependent branch from master afterwards. Recovery for a branch already stacked: `git rebase origin/master` locally (git drops the patches master already has) and note that commit; on the original branch `git merge origin/master`, resolve with `git read-tree <rebased>` then `git checkout-index -af`, confirm `git diff <rebased> HEAD` is empty, commit the merge, and push normally. After merging master into a PR branch, diff the result against master looking for the same fact bound twice, because a merge keeps both sides' bindings and every check still passes. Then run the repo's checks on the merged tree: a name one side deleted and the other began to use merges without a conflict and does not compile.
- PR descriptions for code or generated-behavior changes should expose residual risk and validation signal for human triage. Prefer concise `## Risk` and `## Validation` sections when they add useful review signal; do not block PR creation solely over missing headings.
- Do not post no-findings update-check comments. Comment on a PR only when there are findings, decisions, user-request context, or useful review signal.
- Link every implementation PR to the tracking issue. For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and put `Closes owner/repo#N` only on the final integration PR so the issue closes after the whole chain lands.
- If the issue was missing when implementation starts, create it before opening PRs and add the issue URL or number back to the dev plan.
- When closing a PR without merging, delete its remote branch unless there is a concrete reason to keep it: `forge pr close <number-or-url-or-branch> -d`.
- Commit messages use plain text. Put longer human-facing tracking or discussion in Forge issues or PRs.

## Clones and Concurrent Agents

Several agents may share one workspace. Two agents branching off the same commit rewrite no files, so git does not object when the second moves HEAD out from under the first — the collision is silent. That risk is about an unsandboxed session sharing a checkout: a sandboxed worker holds nothing but its own clone and cannot reach a sibling's tree to collide with it.

- Never `git switch -c agent/<desc>` in a shared checkout. `allod change begin -d <desc> <repo>` is the whole answer for every repo; `record` and `submit` run via `allod change run` against the path it prints.
- A tree `begin -d` makes under `~/changes` is a self-contained clone, not a linked worktree: its objects are copied from the parent, never hardlinked (a hardlink shares the parent's own inode, and the worker could `chmod` or overwrite through it from inside its clone) and never shared through alternates (a parent `git gc` can prune what that still references); the parent repository ledgers the clone under its own `.git`. `allod change list` reads that ledger and reports each clone, plus any linked worktree left from before this change, with the state blocking its removal: clones can be `missing`, `running`, `unsubmitted`, `open`, `merged`, `closed`, or `unknown`, read from the forge rather than the clone's disk; legacy worktrees retain `prunable`, `locked`, `detached`, and `submodule`. Nothing is ever reclaimed implicitly — no local signal distinguishes a dead agent from a working one, so orphans are named, never collected.
- Nobody outside a worker's own sandbox runs git in its clone: `list`, `record`, `submit`, `cleanup`, and `run` judge a clone from the forge or the ledger, never from its tracked config, hooks, or disk state. `run` still carries its own preflight, each with its own named exit code: a configured netrc the invoking user can read, or `~/.claude`, `~/.codex`, `~/.pi`, `~/.claude.json`, or `~/.gitconfig` itself being a symlink — and it gives the sandboxed command no controlling terminal of its own.
- `allod change run` refuses to start, with exit code 12, when the invoking user is one of Nix's own trusted users: the daemon accepts privileged build options from a trusted uid, including a root-run `post-build-hook`, so binding its socket into the sandbox would hand a worker root on the VM. `allod/vm#12` removes that trust fleet-wide; until it lands everywhere, a swarm needs an untrusted invoking user.
- `allod change cleanup` asks the forge for the pull request whose head is the clone's branch, and removes the clone and drops its ledger entry once that pull request is merged or closed; it refuses with no pull request found or an open one, and runs no git command in the clone. A clone whose directory is already gone (`missing`) skips the forge check and drops its ledger entry directly. `run` holds a lock on each clone's ledger file for its lifetime; `list`, `record`, `submit`, and `cleanup` refuse to act on that clone while the lock is held.
- A clone a worker holds is hands-off until the worker exits: `list` shows it `running`; `cleanup` refuses it with exit code 13 rather than wait, so there is nothing to retry — the lock releases with the sandboxed process, and the worker calls `record` and `submit` itself from inside, where the lock is already held. Each run is a user scope named `allod-run-<branch>-<suffix>`, so `systemctl --user list-units 'allod-run-*'` names every live worker by branch, and `systemctl --user show <unit> -p CPUUsageNSec -p MemoryCurrent` is a liveness signal that needs no runner-specific log: CPU time that stops growing is a stall. Inside, `TMPDIR` is a private tmpfs and `~/changes` holds only the named clones, so a brief or a report crosses the boundary on stdin, inline in the prompt, or inside the clone, never as a file elsewhere.
- Inside the sandbox, `codex` starts with `WARNING: proceeding, even though we could not create PATH aliases: Read-only file system` because `~/.codex` is bound read-only; it is noise, not a failure.
- Default-branch commits stay in place. Git refuses one branch in two worktrees (`fatal: '<branch>' is already used by worktree at <path>`), so that flow cannot be isolated; its safety net is the push rejection — `git pull --rebase`, push again. Automating that recovery is deferred: `allod/tools` issue #124.
- Name your files when recording in a shared checkout: `record` stages `git add -u`, which sweeps another agent's tracked edits. Untracked files are skipped. `allod/tools` issue #118.
- `protected-refs-policy` resolves a worktree to the repository it belongs to, so `protected-branches`, `signing-required-branches`, and repo-local hooks apply inside one; `agent/*` work there is not blocked. A machine has that hook only after its `allod/archetypes` pin carries it and home-manager has rebuilt: `grep -c remote_identity ~/.config/git/protected-refs-policy` prints 0 on the older hook, which keys on path alone and is blind inside every worktree. Inside a sandboxed clone the checkout path matches no entry, so the hook falls through to the origin match instead and still blocks the protected branch.

## Branch Protection

The forge is the authoritative wall for an account without push rights on the branch: framework repo default branches are protected server-side, so an agent's direct push is rejected no matter what the local rails do. For that account `protected-branches` and the `protected-refs-policy` hook are backstops that fail earlier and more legibly, and a local rail falling open is a defence-in-depth gap rather than a path to publication.

Branch protection does not restrain a repo admin, who can push to a protected default branch directly and may do so routinely. Never offer the forge as the thing that will catch a maintainer's mistaken direct push, and do not reason about their workflow from the agent's constraint: for them the local rails are the only rails, and `modules/agent-hooks.nix` is imported by `mkDevVm` alone, so on a hypervisor there are none.

`protected-branches` is a hand-maintained mirror of the forge rules and can drift from them silently.

- Check effective status: `GET /api/v1/repos/{owner}/{repo}/branches/{branch}` returns `protected` and `user_can_push`.
- Enumerating or changing the rules needs repo admin, which agents do not have; `/branch_protections` returns 403. Changing protection is a human act at the forge.
- `forge` has no branch-protection command yet.

## Commit Messages

The tracked `commit-msg` hook (`archetypes/hooks/commit-msg`) rejects:

- A word-bounded `close[sd]?`, `fix(e[sd])?`, or `resolve[sd]?` before a `#<digit>` on the same line — innocent prose trips it ("the resolved target … owner/repo#56"). Use `Refs`, `Repair commit: <hash>`, `Amended in <hash>`, or split the word and the reference across lines. `fixture` and `prefix` are fine.
- Agent attribution trailers (`Co-Authored-By: Claude`).
- An inexact model footer; use the exact slug, e.g. `Model: gpt-5.5`.

## Forge CLI

- Prefer `allod change submit` for PR creation.
- Edits: `forge pr edit <number> [--title <title>] [--body <text> | --body-file <file>]`
- Comments: `forge pr comment <number> [--body <text> | --body-file <file>]`
- Close abandoned PRs with branch cleanup: `forge pr close <target> -d`
- All `forge` content commands accept `-b` or `--body` and `-F` or `--body-file`; use `--body-file -` for stdin.
- `--body` does not interpret `\n`, and backticks inside a double-quoted `--body` are command-substituted by the shell before `forge` ever sees them, silently deleting the code span's contents. Forge has no comment edit, so a mangled comment is permanent. Always pass Markdown through `--body-file` — a file, or real lines piped to `--body-file -`.
- Issue and PR bodies should use prose as single long lines; only break at paragraph boundaries, list items, and code blocks.

## Merge Rights

Agents hold merge rights on a repository only where the owner has granted them at the forge, and use them only through the `verify-merge` skill, by residual risk class (`risk.md`). The forge enforces the account split: the author account has no merge right anywhere, the verifier account sits on the merge whitelist of the repositories in the ladder, and `forge pr merge` pins the head commit it verified and reads the base branch back after merging. The class limit, the cold-session rule and the rung are prose; the proof comment on every merge is how the owner sees them kept, and the digest is the review.

The ladder, per repository and class:

- R0 and R1: the verifier merges after posting proof.
- R2: the cross-family reviewer pass and one fix round (`risk.md`), then proof, then a 24-hour veto window before the merge unless the ledger comment waives it.
- R3 and up, and any multi-repo change: human gate, as before. The verifier still posts the proof.

A rung is `agent merge` or `human gate`, per class per repository, set by the owner's latest comment on that repository's ledger issue; private memory names the issue. A repository with no feature map (`testing.md`, Feature Map) is proved by its checks plus the PR body's own Validation commands, rerun by the verifier in its clone; the map is what lets a verifier drive beyond what the author ran. Promotion: a class earns the next rung after one week on its current rung with no revert and no coverage gap in the digest. Demotion: any revert of an agent merge puts that class back on human gate at once, by the owner's comment. Probation starts at R0 and R1 on one repository; nothing is granted wholesale, and the human gate is always one comment away.

## Landing a PR

The pull request opens last, when the change is ready for the verifier, so the branch on the forge is never rewritten: force-push on `agent/*` stays refused and a PR branch only ever gains commits.

- **Build, review and tidy before pushing.** The worker implements and commits in its clone and pushes nothing. The overseer verifies the work and runs the cold review the score requires (`risk.md`) on the diff; the fix round goes back to the same clone, still unpushed. The worker then tidies its history into logical commits, with a non-interactive rebase in the clone, and only then runs `allod change record` and `allod change submit`.
- **A cold verifier merges.** The overseer starts a fresh agent holding none of the driving session's context (a subagent with a fresh context is cold; the account split is the wall). Its brief says: the PR as `owner/repo#N`; that it is the verifier and selects the verifier credential with `FORGE_TOKEN_FILE` (private memory names the file); read memory, then `allod skill verify-merge` and follow it step by step, stopping where it stops; work only in a scratch clone it makes itself, never under the work directory or `~/changes`; `--rebase`, never `--squash` or `--merge`; report back the lead it posted, its decision with the one deciding fact, and the merge commit and revert line. Relay that report to the owner verbatim; a stop is the owner's to resolve, not the driver's to retry.
- **A verifier's finding is one more commit.** It lands on the open pull request as its own logical commit, and the verifier starts over on the new head.
- **Afterwards**, `allod change cleanup` on the change's clones once the PR is merged.

The owner's window is `skills/verify-merge/merge-digest`, run on demand and at the week review. The revert is the line the landing comment carries, run by the owner on master; agents never push a default branch. Everything above the ladder keeps the merge gate in `memory.md` (PR Workflow): the owner restating the owner summary.
