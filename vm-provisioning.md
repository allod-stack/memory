# VM Provisioning Stack

Repos: `archetypes` for the VM framework (archetype merge, builders, shared modules), `profiles` for the example machine profile definitions it composes, `vm` for shared NixOS modules, `nexus` for provisioning scripts, and `inventory` for machine inventory.

## The Public Data Repos Are Templates, Keys Included

`inventory`, `secrets`, `profiles` and `deploy` in the public org are templates. A deployment forks them, redirects the three data inputs, and builds real machines from the fork. Nothing in the public copies describes a running machine.

That holds for key material too, and it is the part that misleads. `secrets/machine-host-keys.json` and `secrets/vm-host-keys/<machine>-ssh.age` look like a live key store, but their keys are synthetic: the registered public key for a machine does not match that machine's real `/etc/ssh/<name>.pub`. Compare the two before believing either.

So never write a real machine's key material into the public repos. Publishing is irreversible, and the only remedy is rotation. A new machine's identity goes in the private fork; what belongs in public is the mechanism that makes the identity optional or shaped, never the identity.

Real-but-public component names — the forge host, the agent account — are correct in the public copies and are not evidence that the surrounding data is real (`architecture.md` principle 5).

## Source of Truth

- Machine platform, type, and hardware: `inventory/flake.nix`
- VM IPs, repos, and Forge keys: `inventory/scripts/vm-specs.json`, derived from the Nix attrset
- Which registry `allod` reads on a dev VM: the machine's `profile.inventoryCheckout` in `inventory/flake.nix`, rendered as `INVENTORY`; unset, `allod` there reads the public `allod/inventory` checkout. It is never set in `vm-specs.json`, which is generated
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
- **SSH host key** - age-encrypted, injected by `nixos-anywhere`; pipe directly, never use command substitution because it strips the trailing newline.
- **Activation scripts must tolerate provisioning** - during `nixos-anywhere`, `TMPDIR` can point to a non-existent path and agenix secrets or optional credentials can be unavailable. In NixOS activation snippets, use a conditional no-op for missing optional resources; do not `exit 0`, because snippets are concatenated into one activation script and that exits the whole activation before `/run/current-system` is linked.
- **Host-key rotation does not re-key the installer image** - `provision-vm` passes the host key to `nixos-anywhere` as both the age identity and the SSH identity for `root@<target>`, so the installer image must already trust that key. Until the image is rebuilt with the rotated key, provisioning fails with `Permission denied (publickey)`. Tracked as `allod/nexus` issues #8 and #9.
- **VMs get virtio-gpu without 3D** - `new-vm` passes no `--video`, so guests boot with `-virgl` and zero cap sets. A GPU PCI ID and a world-readable render node exist anyway, so read `dmesg | grep 'features:'` before claiming a VM has 3D. EGL needs `hardware.graphics.enable`, which is off fleet-wide and buys llvmpipe software rendering for about 229 MiB.
- **`error: unexpected end-of-file` is a truncated fetch, not a flake fault** - it is nix's message for a stream that ended early, and in a provisioning run it means a NAR download was cut mid-stream. A retry is the fix; a lock update is not, and neither is any change to the flake. `provision-vm`'s `on_error` trap prints `${BASH_LINENO[0]}`, the first line of the continued `nixos-anywhere` invocation, so every failure in the disko or install phase reports that one line and names no phase. Read the run's `###` banners to find the step, and settle whether evaluation is involved at all with `nix eval <flake>#nixosConfigurations.<name>.config.system.build.toplevel.drvPath`.
- **The install phase re-downloads a closure the host already built** - with `--phases disko,install` nixos-anywhere builds on the hypervisor, then its default `--substitute-on-destination`, plus the machine's own substituters appended to the installer's `~/.config/nix/nix.conf`, make the installer fetch each path from the cache instead of taking it across the bridge. A dev VM closure is about 4.2 GiB over roughly 1300 paths, sent one NAR per round trip with no progress total, so the step is long and looks hung; `nix copy` does not loop. `allod/nexus#69` carries the fix, a single `--no-substitute-on-destination`.

## Checkout Paths Are Load-Bearing

`protected-refs-policy` and `allod change` look a repo up by its `$HOME`-relative checkout path first, then by the `owner/repo` its `origin` names. A protected repo checked out anywhere other than its listed path is refused, not unprotected: `allod change begin` and `record` exit 8 on any branch, the hook blocks the protected branch, and both name the expected and the actual path. Keep checkouts at the exact registry path. A repo whose origin matches no entry, or that has no origin, is unprotected silently. A machine whose hook predates this (`git-workflow.md`, Worktrees and Concurrent Agents) still falls open on a misplaced checkout; the push is refused forge-side either way.

`flake-update-cascade` follows the same rule through the shared `internal/protection` package, and stops before touching anything on a misplaced checkout or on a list it cannot read. It also still honours the key `work/<repo>` wherever `WORK_DIR` puts the workspace. It gives a verdict only for repositories the run would touch.

The installed `allod` and `flake-update-cascade` are wrappers that put the real `git` first on `PATH`, so a test suite with a stand-in `git` fails against them from its first case. Point `ALLOD_UNDER_TEST` and `CASCADE_UNDER_TEST` at the `.…-wrapped` binary beside the wrapper, and search that file, not the wrapper, when checking what a build contains.

## Privacy VMs

- No Forge or git by default; rebuild from the host for config changes.
- Strip UTF-8 BOM from external JSON before piping to `jq` when an upstream file includes one: `sed 's/^\xef\xbb\xbf//'`.
- Tor-only VMs must keep traffic policy in the VM configuration, not in ad hoc runtime commands.
