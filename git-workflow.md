# Git Workflow

- Before starting work: `work-diff` -> `pull-all`.
- Use `allod change` for change work instead of manual git and Forge mutation steps:
  - Start protected repo work with `path=$(allod change begin -d <short-description> <repo-path>)`, then work in `$path`.
  - Commit and push with `allod change record -m <message> [-f <file>...]`.
  - Open PRs with `allod change submit -t <title> -F <body-file>`.
  - Remove merged worktrees with `allod change cleanup <worktree-path>`.
- `allod change begin` reads `~/.config/git/protected-branches`; protected repos get temporary worktrees and `agent/<description>` branches.
- Unprotected repo plus PR requested: create or switch to `agent/<short-description>` from `master`, then use `allod change record` and `allod change submit`.
- No external remote pushes unless the remote is explicitly allowed locally.
- PR branches: add commits, do not force-push.
- PR descriptions for code or generated-behavior changes must include `## Validation` with exact reviewer commands. Docs-only PRs do not need validation sections unless a command checks a real failure mode.
- Do not post no-findings update-check comments. Comment on a PR only when there are findings, decisions, user-request context, or useful review signal.
- Link every implementation PR to the tracking issue. For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and put `Closes owner/repo#N` only on the final integration PR so the issue closes after the whole chain lands.
- If the issue was missing when implementation starts, create it before opening PRs and add the issue URL or number back to the dev plan.
- When closing a PR without merging, delete its remote branch unless there is a concrete reason to keep it: `forge pr close <number-or-url-or-branch> -d`.
- Commit messages use plain text. Put longer human-facing tracking or discussion in Forge issues or PRs.

## Forge CLI

- Prefer `allod change submit` for PR creation.
- Edits: `forge pr edit <number> [--title <title>] [--body <text> | --body-file <file>]`
- Comments: `forge pr comment <number> [--body <text> | --body-file <file>]`
- Close abandoned PRs with branch cleanup: `forge pr close <target> -d`
- All `forge` content commands accept `-b` or `--body` and `-F` or `--body-file`; use `--body-file -` for stdin.
- `--body` does not interpret `\n`; for multiline Markdown, pipe real lines to `--body-file -` or pass a file.
- Issue and PR bodies should use prose as single long lines; only break at paragraph boundaries, list items, and code blocks.
