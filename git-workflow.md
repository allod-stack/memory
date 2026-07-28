# Git Workflow

- Before starting work: `work-diff` -> `pull-all`.
- Before treating a requested Allod file as missing or stale, ensure its repo is
  current. If `pull-all` fails there, inspect the repo: with a clean worktree on
  a gone upstream, switch to the default branch and fast-forward pull; otherwise
  read the file from `origin/HEAD` or state the freshness problem.
- Use `allod change` for change work instead of manual git and Forge mutation steps:
  - Start branch work with `path=$(allod change begin -d <short-description> <repo-path>)`, then work in `$path`.
  - Commit and push with `allod change record -m <message> [-f <file>...]`.
  - Open PRs with `allod change submit -t <title> -F <body-file>`.
  - Find reclaimable worktrees with `allod change list [<repo-path>]`.
  - Remove merged worktrees with `allod change cleanup <worktree-path>`.
- `-d` is the isolation switch: it creates a worktree under `~/changes` and an `agent/<description>` branch for every repo, protected or not. Without `-d`, `begin` prints the shared checkout path and creates nothing — the in-place flow for a repo's default branch, which protected repos refuse.
- `protected-branches` governs only which branch is protected, never whether a change is isolated.
- Branch work always happens in a worktree, never by switching branches in a shared checkout — see Worktrees and Concurrent Agents.
- No external remote pushes unless the remote is explicitly allowed locally.
- PR branches: add commits, do not force-push.
- PR descriptions for code or generated-behavior changes should expose residual risk and validation signal for human triage. Prefer concise `## Risk` and `## Validation` sections when they add useful review signal; do not block PR creation solely over missing headings.
- Do not post no-findings update-check comments. Comment on a PR only when there are findings, decisions, user-request context, or useful review signal.
- Link every implementation PR to the tracking issue. For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and put `Closes owner/repo#N` only on the final integration PR so the issue closes after the whole chain lands.
- If the issue was missing when implementation starts, create it before opening PRs and add the issue URL or number back to the dev plan.
- When closing a PR without merging, delete its remote branch unless there is a concrete reason to keep it: `forge pr close <number-or-url-or-branch> -d`.
- Commit messages use plain text. Put longer human-facing tracking or discussion in Forge issues or PRs.

## Worktrees and Concurrent Agents

Several agents may share one workspace. Two agents branching off the same commit rewrite no files, so git does not object when the second moves HEAD out from under the first — the collision is silent.

- Never `git switch -c agent/<desc>` in a shared checkout. `allod change begin -d <desc> <repo>` is the whole answer for every repo; `record` and `submit` from inside the path it prints.
- `allod change list` reports each linked worktree with the one thing blocking its removal: `prunable`, `locked`, `detached`, `submodule`, `dirty`, `unpushed`, or `clean`. `clean` means `cleanup` will succeed. Nothing is ever reclaimed implicitly — no local signal distinguishes a dead agent from a working one, so orphans are named, never collected.
- `allod change cleanup` deletes the `agent/*` branch, and its unpushed-commit guard answers "nothing unpushed" when it cannot resolve a remote base — a repo with no `origin`, say — so it can destroy commits that exist nowhere else while `list` calls that worktree `clean`. `allod/tools` issue #127.
- Default-branch commits stay in place. Git refuses one branch in two worktrees (`fatal: '<branch>' is already used by worktree at <path>`), so that flow cannot be isolated; its safety net is the push rejection — `git pull --rebase`, push again. Automating that recovery is deferred: `allod/tools` issue #124.
- Name your files when recording in a shared checkout: `record` stages `git add -u`, which sweeps another agent's tracked edits. Untracked files are skipped. `allod/tools` issue #118.
- Path-keyed hook policies are blind inside a worktree: `protected-refs-policy` keys `protected-branches` and `signing-required-branches` off the `$HOME`-relative path, which never matches there. Remote-keyed and branch-name-keyed rules still fire, `record` still checks, and the forge-side wall below is unaffected. Repo-local hooks also do not run in a worktree — `run_repo_hook` looks for `$repo/.git/hooks/`, and a worktree's `.git` is a file; tracked `.hooks` are unaffected. `allod/tools` issue #112.

## Branch Protection

The forge is the authoritative wall: framework repo default branches are protected server-side, so a direct push is rejected no matter what the local rails do. `protected-branches` and the `protected-refs-policy` hook are backstops that fail earlier and more legibly — a local rail falling open is a defence-in-depth gap, not a path to publication. `protected-branches` is a hand-maintained mirror of the forge rules and can drift from them silently.

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
