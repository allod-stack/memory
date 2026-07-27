# Git Workflow

- Before starting work: `work-diff` -> `pull-all`.
- Before treating a requested Allod file as missing or stale, ensure its repo is
  current. If `pull-all` fails there, inspect the repo: with a clean worktree on
  a gone upstream, switch to the default branch and fast-forward pull; otherwise
  read the file from `origin/HEAD` or state the freshness problem.
- Use `allod change` for change work instead of manual git and Forge mutation steps:
  - Start protected repo work with `path=$(allod change begin -d <short-description> <repo-path>)`, then work in `$path`.
  - Commit and push with `allod change record -m <message> [-f <file>...]`.
  - Open PRs with `allod change submit -t <title> -F <body-file>`.
  - Remove merged worktrees with `allod change cleanup <worktree-path>`.
- `allod change begin` reads `~/.config/git/protected-branches`; protected repos get temporary worktrees and `agent/<description>` branches.
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

- Never `git switch -c agent/<desc>` in a shared checkout. `allod change begin` only creates a worktree for repos in `protected-branches`; for the rest create one yourself — `git -C <repo> worktree add ~/changes/<desc> -b agent/<desc> origin/<default>` — then `record` and `submit` from inside it. Unconditional isolation: `allod/tools` issue #116.
- Default-branch commits stay in place. Git refuses one branch in two worktrees (`fatal: '<branch>' is already used by worktree at <path>`), so that flow cannot be isolated; its safety net is the push rejection — `git pull --rebase`, push again.
- Name your files when recording in a shared checkout: `record` stages `git add -u`, which sweeps another agent's tracked edits. Untracked files are skipped. `allod/tools` issue #118.
- Hooks are blind inside a worktree: `protected-refs-policy` keys off the `$HOME`-relative path, which never matches there, so branch-protection and signing guards are silent. `record` still checks. `allod/tools` issue #112.

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
- `--body` does not interpret `\n`; for multiline Markdown, pipe real lines to `--body-file -` or pass a file.
- Issue and PR bodies should use prose as single long lines; only break at paragraph boundaries, list items, and code blocks.
