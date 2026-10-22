# Git Workflow

- Before starting work: `work-diff` -> `pull-all`.
- Before treating a requested Allod file as missing or stale, ensure its repo is
  current. If `pull-all` fails there, inspect the repo: with a clean worktree on
  a gone upstream, switch to the default branch and fast-forward pull; otherwise
  read the file from `origin/HEAD` or state the freshness problem.
- Use `allod change` for change work instead of manual git and Forge mutation steps:
  - Start branch work with `path=$(allod change begin -d <short-description> <repo-path>)`, then work in `$path`.
  - Commit and push with `allod change record -m <message> [-f <file>...]`. A multi-line message comes from a file with `-M <file>`, or from stdin with `-M -`; `record` has no `-F`, which is `submit`'s flag for the PR body.
  - Open PRs with `allod change submit -t <title> -F <body-file>`.
  - Find reclaimable worktrees with `allod change list [<repo-path>]`.
  - Remove merged worktrees with `allod change cleanup <worktree-path>`.
- `-d` is the isolation switch: it creates a self-contained clone under `~/changes` and an `agent/<description>` branch for every repo, protected or not. Without `-d`, `begin` prints the shared checkout path and creates nothing — the in-place flow for a repo's default branch, which protected repos refuse.
- `allod patch receive <vm>:<repo-id> <repo-id>` resolves a registry-id source to the repo's shared checkout (`~/work/<checkout>`) on the source machine, never to an `allod change` clone under `~/changes` — a change made in a clone must name the source by path (`<vm>:~/changes/<clone>`), or the relay resolves to the shared checkout, finds no unpushed commits there, and fails with an empty export range (exit 11).
- `allod patch fetch --base <ref>` resolves the ref on the source machine and exits with `base <ref> is not an ancestor of HEAD` once the source checkout's master has moved past the branch point; name the branch point commit instead. Into a machine that cannot SSH out (a dev VM), the relay is `fetch --output <dir>` on nexus, `scp -r` of that artifact directory to the VM, and `allod patch apply <dir> --repo <clone>` there. Applying onto a clone cut from current master is the rebase: `git am --3way` lands the commit on the new base, and the conflict markers it leaves are inside heredocs that `bash -n` never sees, so grep for them before amending.
- `allod change submit` opens the PR but pushes nothing; only `record` pushes. A commit made another way — a relayed patch, an amend after a rebase — needs `git push -u origin <branch>` first, or submit fails with a bare `POST /repos/<owner>/<repo>/pulls failed: HTTP 404`.
- `protected-branches` governs only which branch is protected, never whether a change is isolated.
- Branch work always happens in its own clone, never by switching branches in a shared checkout — see Worktrees and Concurrent Agents.
- No external remote pushes unless the remote is explicitly allowed locally.
- PR branches: add commits, do not force-push.
- Do not stack a PR on another open PR. PRs land by rebase-merge, so master carries new hashes, and the forge refuses a force-push on `agent/*` (`force-push is not permitted ... Add commits instead`): the stacked branch conflicts the moment its base merges, and the local rebase that would fix it cannot be pushed. Wait for the base to merge, or cut the dependent branch from master afterwards. Recovery for a branch already stacked: `git rebase origin/master` locally (git drops the patches master already has) and note that commit; on the original branch `git merge origin/master`, resolve with `git read-tree <rebased>` then `git checkout-index -af`, confirm `git diff <rebased> HEAD` is empty, commit the merge, and push normally. After merging master into a PR branch, diff the result against master looking for the same fact bound twice, because a merge keeps both sides' bindings and every check still passes. Then run the repo's checks on the merged tree: a name one side deleted and the other began to use merges without a conflict and does not compile.
- PR descriptions for code or generated-behavior changes should expose residual risk and validation signal for human triage. Prefer concise `## Risk` and `## Validation` sections when they add useful review signal; do not block PR creation solely over missing headings.
- Do not post no-findings update-check comments. Comment on a PR only when there are findings, decisions, user-request context, or useful review signal.
- Link every implementation PR to the tracking issue. For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and put `Closes owner/repo#N` only on the final integration PR so the issue closes after the whole chain lands.
- If the issue was missing when implementation starts, create it before opening PRs and add the issue URL or number back to the dev plan.
- When closing a PR without merging, delete its remote branch unless there is a concrete reason to keep it: `forge pr close <number-or-url-or-branch> -d`.
- Commit messages use plain text. Put longer human-facing tracking or discussion in Forge issues or PRs.

## Worktrees and Concurrent Agents

Several agents may share one workspace. Two agents branching off the same commit rewrite no files, so git does not object when the second moves HEAD out from under the first — the collision is silent. That risk is about an unsandboxed session sharing a checkout: a sandboxed worker holds nothing but its own clone and cannot reach a sibling's tree to collide with it.

- Never `git switch -c agent/<desc>` in a shared checkout. `allod change begin -d <desc> <repo>` is the whole answer for every repo; `record` and `submit` from inside the path it prints.
- A tree `begin -d` makes under `~/changes` is a self-contained clone, not a linked worktree: its objects are copied from the parent, never hardlinked (a hardlink shares the parent's own inode, and the worker could `chmod` or overwrite through it from inside its clone) and never shared through alternates (a parent `git gc` can prune what that still references); the parent repository ledgers the clone under its own `.git`. `allod change list` reads that ledger and reports each clone, plus any linked worktree left from before this change, with the state blocking its removal: clones can be `missing`, `running`, `unsafe`, `moved`, `dirty`, `unpushed`, or `clean`; legacy worktrees retain `prunable`, `locked`, `detached`, and `submodule`. `clean` has no local blocker; `cleanup` still fetches and can refuse if the remote is unavailable. Nothing is ever reclaimed implicitly — no local signal distinguishes a dead agent from a working one, so orphans are named, never collected.
- The overseer never runs git against a clone that could run a command or hide its own state from the gate. `list`, `record`, `submit`, `cleanup`, and `run` all read the clone's config with git's file-only reader before their first real git command against it, and refuse with exit code 11 if `.git` is missing or not an ordinary directory, the clone has a populated submodule (sandboxed clones do not support them at all), its config carries a key outside a fixed allowlist (the `core.*` keys and `remote.origin.*` an ordinary clone has, plus `branch.<ledger-branch>.remote`/`.merge` pinned to exactly the shape `begin` sets — every other `branch.*` key, and any `extensions.*` beyond `refstorage`/`objectformat`, is refused), an executable `.git/hooks` entry not named `*.sample`, or a `.git/config.worktree` exists. `list` never dies on a failing clone: it marks that row `unsafe`, runs no git against it, and moves on to the next one. `run` carries its own extra preflight beyond the gate, each with its own named exit code: a configured netrc the invoking user can read, or `~/.claude`, `~/.codex`, `~/.pi`, `~/.claude.json`, or `~/.gitconfig` itself being a symlink — and it gives the sandboxed command no controlling terminal of its own.
- Host-side Git commands on a ledgered clone use isolated configuration: no global or system hooks, filters, credential helpers, or caller-supplied `GIT_*` settings. They carry the overseer's Git identity. An HTTPS origin that needs a global credential helper needs another approved transport before `record` or `cleanup` can contact it.
- `allod change run` refuses to start, with exit code 12, when the invoking user is one of Nix's own trusted users: the daemon accepts privileged build options from a trusted uid, including a root-run `post-build-hook`, so binding its socket into the sandbox would hand a worker root on the VM. `allod/vm#12` removes that trust fleet-wide; until it lands everywhere, a swarm needs an untrusted invoking user.
- `allod change cleanup` fetches (`git fetch --prune origin`) inside the clone before judging unpushed commits, and refuses if that fetch itself fails; once the tree is clean and nothing is unpushed it removes the clone and drops its ledger entry. A clone always has an `origin` — `begin -d` sets it — so the unpushed check no longer falls back to reading an unresolvable remote as "nothing unpushed"; it refuses and asks for inspection instead, closing the destructive case `allod/tools` issue #127 named for the old worktree model. A clone whose directory is already gone (`missing`) skips all of that and drops its ledger entry directly, since there is no admin side-channel left to clear first. `run` holds a lock on each clone's ledger file for its lifetime; `list`, `record`, `submit`, and `cleanup` refuse to act on that clone while the lock is held.
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

The owner's window is `skills/verify-merge/merge-digest`, run on demand and at the week review. The revert is the line the landing comment carries, run by the owner on master; agents never push a default branch. Everything above the ladder keeps the merge gate in `memory.md` (PR Workflow): the owner restating the owner summary.
