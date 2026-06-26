# Allod

Self-sovereign NixOS VM stack for agentic coding and privacy tasks.

## Forgejo Org

- Org: `forge.anarch.diy/allod` - public repos
- Private or identity-bearing repos stay outside the public org.
- Local path convention: `<workspace>/allod/<repo>`

## Allod Directory Conventions

- Archive files under `allod/strategy/archive/<type>/`.
- Each `allod/*` directory is its own git repo.

## Repo Inventory

| Repo | Local path | Notes |
|------|------------|-------|
| `allod/tools` | `allod/tools` | General-purpose CLI tools |
| `allod/memory` | `allod/memory` | Public Allod workflow memory |
| `profiles` | `allod/profiles` | NixOS VM configs |
| `vm` | `allod/vm` | VM shared modules and bootstrap |
| `nexus` | `allod/nexus` | Host machine NixOS config and provisioning scripts |
| `inventory` | `allod/inventory` | Machine platform, type, hardware, and VM specs |
| `secrets` | `allod/secrets` | Encrypted secrets and recipient metadata |
| `strategy` | `allod/strategy` | Development plans and design notes. Archive: `strategy/archive/<type>/` |

Audit rule before migrating a repo to the public org: no plaintext secrets, no personal identifiers, and no host-specific private addressing.
Age-encrypted blobs are fine in public repos by design.

## forge CLI

`forge` is the primary interface for working with Forgejo PRs and issues.
Run `forge --help` for full usage.

Mutation commands follow the common content-flag shape:

```bash
forge pr create --title <title> [--head <branch>] [--base <branch>] \
  [--body <text> | --body-file <file>]
forge pr edit <number> [--title <title>] [--body <text> | --body-file <file>]
forge pr comment <number> [--body <text> | --body-file <file>]
forge pr close {<number> | <url> | <branch>} [-c <text>] -d
forge issue create --title <title> [--body <text> | --body-file <file>]
forge issue edit <number> [--title <title>] [--body <text> | --body-file <file>]
```

Short aliases match `gh`: `-t`, `-b`, `-F`, `-H`, and `-B`.
`--body-file -` reads stdin. `pr create` defaults the head to the current branch and the base to the repository default branch.
Use `-d` when closing abandoned PRs so the remote head branch is deleted.

## Workspace Tools

Session: `work-diff` -> `pull-all` -> `allod change begin` -> work -> `allod change record` -> `allod change submit`

- `pull-all` - sync all repos at session start; skips dirty or unpushed repos
- `work-diff` - show uncommitted changes across all repos
- `allod change begin -d <desc> <repo>` - create a protected-repo worktree and branch when required
- `allod change record -m <msg> [-f <file>...]` - stage, commit, and push additively
- `allod change submit -t <title> -F <body-file>` - create a PR; body must include `## Validation`
- `allod change cleanup <worktree>` - remove a clean merged worktree and local `agent/*` branch
- `flake-status [input] [--check-upstream]` - inspect flake lock staleness before updates
