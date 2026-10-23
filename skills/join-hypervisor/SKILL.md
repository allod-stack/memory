---
name: join-hypervisor
summary: Finish a joining hypervisor's declaration from its join branches
description: Complete a second hypervisor's inventory, secrets, and profiles declaration from its join branches. Use when the owner's brief "join-hypervisor <name>" arrives, or when a join/<name> branch appears on inventory or secrets.
---

# join-hypervisor

`allod/deploy`'s `join-deployment.sh`, run on a fresh machine, pushes a
`join/<name>` branch to the inventory and secrets checkouts and prints
`join-hypervisor <name>`. This skill is what completes that declaration: one
pull request per data repository, each carrying only the facts those branches
already hold, so no running machine changes. It stops once the PRs are open;
the owner reviews, merges, and runs `admit-hypervisor <name>` on the primary.

## The facts come from the branches

In the inventory and secrets checkouts, pull the branch:

```
git fetch origin join/<name> && git checkout join/<name>
```

Read exactly two facts, both already on these branches:

- `hosts/<name>/hardware.nix` in inventory — the new machine's hardware
  module. Its platform (the Nix system string it sets, e.g.
  `nixpkgs.hostPlatform`) is the inventory entry's `platform`.
- `machine-host-keys.json["<name>"]` in secrets — `{active, staged}`, its
  host key material.

Never ask the owner for a key, a token, or a hostname, and never read or
print either secret's value; only the public key material already committed
to the branch is used.

## Inventory PR (adds to join/<name>, inventory checkout)

- `machines.<name>` in `flake.nix`: `type = "hypervisor"`; `platform` from
  the hardware module; `hardware = ./hosts/<name>/hardware.nix`; `repos`,
  the same set the primary's hypervisor entry lists; no guests and no
  `host`.
- `host = "<primary>"` on every existing guest machine — the inventory
  refuses to evaluate with two hypervisors and a guest that names neither.
- Regenerate what the inventory derives from `machines`:
  `nix eval .#lib.vmSpecsJson --raw | jq -S . > scripts/vm-specs.json`.
  Diff it against the previous committed copy and confirm it differs only by
  the `host` field on each guest.

## Secrets PR (adds to join/<name>, secrets checkout; the key is already there)

- `lib.hypervisorIdentities.<name>` in `flake.nix`: `sshPublicKeys` built
  from `machine-host-keys.json["<name>"]` (its `active` key, plus `staged`
  when set); every other field — `username`, `forgeHost`, `forgePort`,
  `userForgejoTokenFile` (a path to the new per-machine token ciphertext,
  not yet created), `siteHostingConfigFile`, `operatorPublicKeys` — copied
  from the primary's own `hypervisorIdentities` entry.
- Any check that pins `hypervisorIdentities` to exactly one entry (for
  example, an assertion that `attrNames hypervisorIdentities` equals a
  one-element list) relaxes to: the primary is present, every identity
  exports `sshPublicKeys`, and no hypervisor name collides with a guest's.
- `allod secret declare user-forgejo-token-<name> --kind user --owner <name>
  --to <name> --deployed-path <path> --verify <command> --service forgejo`
  — appends the credential `pending`, with a `nixos-host` target naming
  `<name>`. Never run `allod secret create` on it.
- Add `<name>` as a target of every host credential the deployment shares
  across hypervisors: edit each one's rotation-registry group to add a
  `nixos-host` target naming `<name>`. Leave the rekey itself to
  `admit-hypervisor`.
- Touch nothing else: not `nexus-host-key.json`, not
  `identity.hostPublicKeys`, not any guest's identity.

## Profiles PR (new `join/<name>` branch, profiles checkout)

- Copy the primary's hypervisor directory, `hosts/hypervisor/<primary>/`, to
  `hosts/hypervisor/<name>/` unchanged — only the directory name differs.
  Nothing in a guest's definition changes.

## Measure each PR

- Each repository's own checks pass first.
- From the deploy checkout, after pushing all three branches:
  `fleet-diff --override inventory=<rev> --override secrets=<rev> --override
  profiles=<rev> --expect-none`, the three revisions being the pushed
  `join/<name>` tips. A clean run is the proof that no running machine
  changes.
- Each PR body states what the owner does next: merge all three, then run
  `admit-hypervisor <name>` on the primary.

## What this skill refuses

Never run `admit-hypervisor`, merge a PR, re-lock the deploy flake, rebuild
anything, hold or print a key or a token, or commit to a default branch.
These are exactly the four things `admit-hypervisor` checks before it will
run: the inventory PR gives it a hypervisor-typed machine in the pinned
inventory; the secrets PR gives it the machine's key in
`machine-host-keys.json`, its export from `lib.hypervisorIdentities`, and its
`user-forgejo-token-<name>` credential declared `pending` with a
`nixos-host` target.
