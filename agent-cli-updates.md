# Coding Agent CLI Updates

## Source and fleet bump

- Dev VMs install the agent CLIs from `pkgs-unstable` in each machine's `home.nix`, unpinned — the version comes from the lock.
- `nixpkgs-unstable` (`github:NixOS/nixpkgs/nixos-unstable`) is declared in `vm/flake.nix` and locks in the deploy flake as node `archetypes/vm/nixpkgs-unstable`.
- Bump the fleet: `nix flake update archetypes/vm/nixpkgs-unstable` in the deploy flake, commit the lock, then a human rebuilds the VM from the host.

## Channel lag and early access

`nixos-unstable` trails npm by one to three days; compare `pkgs/by-name/cl/claude-code/manifest.json` on `nixos-unstable` against `master`. For a newer version without a rebuild:

```bash
NIXPKGS_ALLOW_UNFREE=1 nix shell --impure github:NixOS/nixpkgs/<rev>#claude-code
nix run github:NixOS/nixpkgs/<rev>#claude-code -- --version   # preview a rev
```

## A new model can require a newer CLI

A model stays absent from the picker until the CLI is new enough — Opus 5 needs claude-code `>= 2.1.219`. When the channel lags, select the model by id and use the no-rebuild path above.

## `yolo` wrapper

`yolo <agent>` (`archetypes/modules/ai-agents.nix`) changes to the workspace root and execs the agent with the fleet's standard flags and appended system prompt. Append `--model <id>` to override its default.
