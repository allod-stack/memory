# VM Tooling Policy

- `jq` belongs on all dev VMs.
- `python3` does not belong on privacy VMs by default. That is policy, not oversight.
- Do not add `python3` to privacy VMs for agent ergonomics; use `jq` for JSON parsing where available.
- Authoritative package lists live in `profiles/hosts/<archetype>/<name>/configuration.nix`.
- No VM carries a Go toolchain. Go tool development uses the owning repo's devShell (`nix develop` in `allod/tools`, repo-pinned) or ad-hoc `nix shell nixpkgs#go`. A go-dev VM profile is deferred until a need a shell cannot serve appears (closure-baked editor tooling, or an egress-restricted VM that cannot reach the nix cache).
