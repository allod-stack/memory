# VM Provisioning Stack

Repos: `archetypes` for the VM framework (archetype merge, builders, shared modules), `profiles` for the example machine profile definitions it composes, `vm` for shared NixOS modules, `nexus` for provisioning scripts, and `inventory` for machine inventory.

## Source of Truth

- Machine platform, type, and hardware: `inventory/flake.nix`
- VM IPs, repos, and Forge keys: `inventory/scripts/vm-specs.json`, derived from the Nix attrset
- Nexus hardware module: `inventory/hosts/nexus/hardware.nix`
- VM profile definitions (per-machine modules): `profiles/hosts/<archetype>/<name>/`; composed by the `archetypes` framework
- Provisioning scripts: `nexus/scripts/`; deploy through the normal flake update and rebuild path
- VM usernames: identity configuration, not `vm-specs.json`
- Forge key secrets: the encrypted secrets repo, not `profiles/secrets/`
- VM SSH match blocks: the encrypted home configuration, not nexus

Architecture strings belong in inventory. Consumers must derive them; do not hardcode them in `archetypes`, `profiles`, checks, scripts, or plans.

## A Runtime or Boot-Path Migration Starts on a Throwaway Machine

The first machine to move onto a new guest runtime or boot path is a purpose-made one that nothing depends on — never the machine the operator develops from. A guest that fails to boot takes its own repair environment with it, and a first-boot or first-rebuild defect is exactly the class these migrations carry. The operator's own dev machine moves only after the throwaway has passed the acceptance tests on real hardware and been rolled back and forward at least once.

Renaming a machine afterwards is not the escape hatch: per-machine encrypted secret filenames are keyed to the name, so re-keying them is a human-only host action. Pick the name before the migration, not after.

The public example carrying the new runtime must therefore be an example and nothing else. A name in the public inventory can also be a real deployed machine whose key material lives in the private secrets repo, and consumers act on the machine fact by name.

## Provisioning Gotchas

- **Host-side only** - provisioning is Nexus-only; dev VMs should not expose provisioning commands.
- **disko replaces hardware-configuration.nix** - disk layout is in `hosts/<vm>/disk.nix`; avoid per-machine UUIDs.
- **agenix secrets in flake.nix inline modules, not configuration.nix** - `nixos-install` runs without `--flake` context.
- **netrc is not git credentials** - see `nix.md` for the three-file netrc layout; `archetypes/modules/netrc.nix` handles conversion.
- **NIX_CONFIG for bootstrap** - `nix.conf` is a read-only store symlink; use `systemctl set-environment NIX_CONFIG="netrc-file = /etc/nix/netrc"` during bootstrap.
- **SSH host key** - age-encrypted, injected by `nixos-anywhere`; pipe directly, never use command substitution because it strips the trailing newline.
- **Activation scripts must tolerate provisioning** - during `nixos-anywhere`, `TMPDIR` can point to a non-existent path and agenix secrets or optional credentials can be unavailable. In NixOS activation snippets, use a conditional no-op for missing optional resources; do not `exit 0`, because snippets are concatenated into one activation script and that exits the whole activation before `/run/current-system` is linked.
- **Host-key rotation does not re-key the installer image** - `provision-vm-from-host` passes the host key to `nixos-anywhere` as both the age identity and the SSH identity for `root@<target>`, so the installer image must already trust that key. Until the image is rebuilt with the rotated key, provisioning fails with `Permission denied (publickey)`. Tracked as `allod/nexus` issues #8 and #9.
- **VMs get virtio-gpu without 3D** - `new-vm` passes no `--video`, so guests boot with `-virgl` and zero cap sets. A GPU PCI ID and a world-readable render node exist anyway, so read `dmesg | grep 'features:'` before claiming a VM has 3D. EGL needs `hardware.graphics.enable`, which is off fleet-wide and buys llvmpipe software rendering for about 229 MiB.
- **`archetypes` `checks.<system>.pi-integration` hard-codes memory checkout aliases** instead of deriving them from the composed `profile.memoryCheckouts`, so a fork re-exporting framework checks may need to filter that one out until `allod/archetypes` issue #11 lands.

## Checkout Paths Are Load-Bearing

`protected-refs-policy` and `allod change` resolve a repo by its `$HOME`-relative checkout path and treat a lookup miss as "not protected", so a repo checked out anywhere other than its registry path silently loses its local rails — no refusal, no hook block. The commit lands locally; the push is still refused forge-side (see `git-workflow.md`), so this costs a manual reset rather than an unwanted publication. Keep checkouts at the exact registry path; a non-canonical layout is a broken guardrail. `allod/tools` issue #112.

## Privacy VMs

- No Forge or git by default; rebuild from the host for config changes.
- Strip UTF-8 BOM from external JSON before piping to `jq` when an upstream file includes one: `sed 's/^\xef\xbb\xbf//'`.
- Tor-only VMs must keep traffic policy in the VM configuration, not in ad hoc runtime commands.
