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

Several agents may share one workspace, so a shared checkout is not yours alone. Two agents branching off the same commit rewrite no files, so git raises no objection when the second one moves HEAD out from under the first — the collision is silent, and neither agent notices until a write lands somewhere it did not expect.

- Never `git switch -c agent/<description>` in a shared checkout. `allod change begin` creates a worktree only for repos listed in `protected-branches`; for any other repo it hands the shared path straight back, so create the worktree yourself with `git -C <repo> worktree add ~/changes/<description> -b agent/<description> origin/<default-branch>` and run `allod change record` and `allod change submit` from inside it. Unconditional isolation is tracked as `allod/tools` issue #116.
- Committing directly to a repo's default branch stays in place. That is the low-friction memory and planning-doc flow, and git cannot isolate it anyway: it refuses to check out one branch in two worktrees (`fatal: '<branch>' is already used by worktree at <path>`). Its collision safety is the push rejection — resolve with `git pull --rebase` and push again.
- Name your files when recording in a shared checkout. `allod change record` stages `git add -u` by default, which picks up every tracked modification in the tree including another agent's. Untracked files are skipped, so new files cannot ride along. Tracked as `allod/tools` issue #118.
- Do not rely on the git hooks inside a worktree. `protected-refs-policy` resolves the repo by its `$HOME`-relative path, which never matches from a linked worktree, so the branch-protection and signing guards are silent there. `allod change record` still performs its own check. Tracked as `allod/tools` issue #112.

## Commit Messages

The tracked `commit-msg` hook (`archetypes/hooks/commit-msg`) rejects three things, so write around them rather than discovering them at commit time:

- A word-bounded `close[sd]?`, `fix(e[sd])?`, or `resolve[sd]?` anywhere before a `#<digit>` on the same line. Innocent prose trips this too, e.g. "the resolved target … owner/repo#56". Use `Refs`, `Repair commit: <hash>`, or `Amended in <hash>`, or keep the word and the issue reference on separate lines. Close issues through PR bodies or manually. `fixture` and `prefix` are fine — the match is word-bounded.
- Agent attribution trailers (`Co-Authored-By: Claude` and friends).
- An inexact model footer. Plan-review commits use the exact model slug, e.g. `Model: gpt-5.5`; check the agent's config or ask rather than guessing.

## Forge CLI

- Prefer `allod change submit` for PR creation.
- Edits: `forge pr edit <number> [--title <title>] [--body <text> | --body-file <file>]`
- Comments: `forge pr comment <number> [--body <text> | --body-file <file>]`
- Close abandoned PRs with branch cleanup: `forge pr close <target> -d`
- All `forge` content commands accept `-b` or `--body` and `-F` or `--body-file`; use `--body-file -` for stdin.
- `--body` does not interpret `\n`; for multiline Markdown, pipe real lines to `--body-file -` or pass a file.
- `forge pr find-by-head <branch>` is broken: it reports an open PR number regardless of the branch queried. So `allod change submit` dies with "PR #N already exists" for a second or stacked PR whenever any other PR is already open in that repo. Work around it with `forge pr create -t <title> -H <branch> -B <base> -F <body-file>` and add any `Depends on: #N` line to the body yourself.
- Issue and PR bodies should use prose as single long lines; only break at paragraph boundaries, list items, and code blocks.
