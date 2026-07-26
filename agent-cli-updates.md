# Coding Agent CLI Updates

How the coding-agent CLIs on dev VMs get their versions, and what to do when a needed version is not on the channel yet.

## Source and fleet bump

- Dev VMs install the agent CLIs from `pkgs-unstable` (`import nixpkgs-unstable`) in each machine's `home.nix`, unpinned — the version comes from the lock, not from a package pin.
- `nixpkgs-unstable` (`github:NixOS/nixpkgs/nixos-unstable`) is declared in `vm/flake.nix` and locks in the deploy flake as node `archetypes/vm/nixpkgs-unstable`.
- To bump the fleet: in the deploy flake checkout run `nix flake update archetypes/vm/nixpkgs-unstable`, commit the lock, then a human rebuilds the VM from the host.

## Channel lag and early access

`nixos-unstable` trails npm by roughly one to three days, so a lock bump only lands a version once the channel carries it. Compare `pkgs/by-name/cl/claude-code/manifest.json` on `nixos-unstable` against `master` to see whether a version has arrived.

To use a newer version without a rebuild:

```bash
NIXPKGS_ALLOW_UNFREE=1 nix shell --impure github:NixOS/nixpkgs/<rev>#claude-code
nix run github:NixOS/nixpkgs/<rev>#claude-code -- --version   # preview a rev
```

Then launch the agent so it runs from `PATH` rather than the profile copy.

## A new model can require a newer CLI

Model availability is gated by CLI version, not only by the API: a model can be absent from the CLI's model picker until the CLI is new enough (for example Opus 5 needs claude-code `>= 2.1.219`). When the channel lags, select the model explicitly by id on the command line and use the no-rebuild path above.

## `yolo` wrapper

The `yolo <agent>` wrapper (`archetypes/modules/ai-agents.nix`) changes to the workspace root and execs the named agent with the fleet's standard flags, including the shared appended system prompt. Append `--model <id>` to override the model the wrapper defaults to.
