# VM Provisioning Stack

Repos: `profiles` for per-VM NixOS configs, `vm` for shared NixOS modules, `nexus` for provisioning scripts, and `inventory` for machine inventory.

## Source of Truth

- Machine platform, type, and hardware: `inventory/flake.nix`
- VM IPs, repos, and Forge keys: `inventory/scripts/vm-specs.json`, derived from the Nix attrset
- Nexus hardware module: `inventory/hosts/nexus/hardware.nix`
- VM NixOS configs: `profiles/hosts/<type>/<vm>/`
- Provisioning scripts: `nexus/scripts/`; deploy through the normal flake update and rebuild path
- VM usernames: identity configuration, not `vm-specs.json`
- Forge key secrets: the encrypted secrets repo, not `profiles/secrets/`
- VM SSH match blocks: the encrypted home configuration, not nexus

Nix system strings such as VM architecture belong in inventory data and `inventory.lib.supportedPlatforms`. Consumers must derive the target system from inventory or from the target NixOS configuration, for example `nixosConfigurations.<vm>.pkgs.stdenv.hostPlatform.system`; do not hardcode architecture strings in profiles, checks, scripts, or review plans.

## Provisioning Gotchas

- **Host-side only** - provisioning is Nexus-only; dev VMs should not expose provisioning commands.
- **disko replaces hardware-configuration.nix** - disk layout is in `hosts/<vm>/disk.nix`; avoid per-machine UUIDs.
- **agenix secrets in flake.nix inline modules, not configuration.nix** - `nixos-install` runs without `--flake` context.
- **netrc is not git credentials** - see `nix.md` for the three-file netrc layout; `profiles/modules/netrc.nix` handles conversion.
- **NIX_CONFIG for bootstrap** - `nix.conf` is a read-only store symlink; use `systemctl set-environment NIX_CONFIG="netrc-file = /etc/nix/netrc"` during bootstrap.
- **SSH host key** - age-encrypted, injected by `nixos-anywhere`; pipe directly, never use command substitution because it strips the trailing newline.
- **Activation scripts must tolerate provisioning** - during `nixos-anywhere`, `TMPDIR` can point to a non-existent path and agenix secrets or optional credentials can be unavailable. In NixOS activation snippets, use a conditional no-op for missing optional resources; do not `exit 0`, because snippets are concatenated into one activation script and that exits the whole activation before `/run/current-system` is linked.

## Privacy VMs

- No Forge or git by default; rebuild from the host for config changes.
- Strip UTF-8 BOM from external JSON before piping to `jq` when an upstream file includes one: `sed 's/^\xef\xbb\xbf//'`.
- Tor-only VMs must keep traffic policy in the VM configuration, not in ad hoc runtime commands.
