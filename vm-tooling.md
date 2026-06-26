# VM Tooling Policy

- `jq` belongs on all dev VMs.
- `python3` does not belong on privacy VMs by default. That is policy, not oversight.
- Do not add `python3` to privacy VMs for agent ergonomics; use `jq` for JSON parsing where available.
- Authoritative package lists live in `profiles/hosts/<category>/<vm>/configuration.nix`.
