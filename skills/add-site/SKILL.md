---
name: add-site
summary: Stand up a site or deck so `allod site view` finds it
description: Create a site or deck repository, register it, and make `allod site view` find it on one dev VM. The preview flake app and site.toml the serve needs, the registry entry with a free preview port, the machine's repo list, and the clone path the registry expects. Use when the owner asks for a deck or a one-page site, when `allod site serve` refuses with "no site.toml", or `allod site view` with "unknown site".
---

# Add a site

A site is previewable by id only when three things agree: the repository carries a
`preview` flake app and a `site.toml`, the inventory's repository registry names it with
a free `preview_port`, and exactly one dev VM lists it in `repos`. The serve and view
contract is `allod/tools` `docs/allod-site-preview.md`; this is the order that gets a new
site there.

## 1. The repository

- `index.html` (one file; a deck keeps its slides, keys and notes in it), `README.md`,
  `flake.nix`, `flake.lock`, `site.toml`.
- `flake.nix`: `packages.<system>.default` copies the file(s) into `$out` after refusing
  symlinks; `apps.<system>.preview` runs
  `live-server --host "$ALLOD_PREVIEW_INTERFACE" --port "$ALLOD_PREVIEW_PORT" .`
  Nothing else. `live-server` has no `--no-browser` flag; the unit dies with
  `unexpected argument` if you add one.
- `site.toml`: `domain = "<name>"`. Required even for a site that will never deploy:
  a directory-inferred `allod site serve` walks up for this marker and refuses without it.
- `nix flake lock` once; commit the lock.

The forge CLI cannot create repositories. Ask the owner for the empty repository with
the agent as collaborator, then `git remote add origin <its ssh clone url>` and push. A
push that fails with "repository exists / access rights" before the owner says the
repository exists is the repository not existing yet, not a key problem: an ssh
connection to the forge's git user shows the key is fine.

## 2. The clone path is the registry's

The registry's `checkout` is the directory under `~/work` on every machine, and
`allod site view` serves from there. Name the clone `~/work/<id>` from the start, or
`mv` it before registering; a preview started from another path is a different systemd
unit (`allod-preview-work-<path>`) that `view` never finds.

## 3. Register it (the inventory repository, a PR)

In a worktree from `allod change begin -d <desc> <inventory checkout>`:

- `scripts/repositories.json` is hand-edited: add
  `"<id>": {source:"forge", remote:"<id>", checkout:"<id>", preview_port:<n>}`.
  Ports in use are in that file; two sites may not share one, and the flake check refuses
  a port outside 1024-65535.
- `flake.nix`: add `"<id>"` to one dev VM's `repos` list, the VM the owner previews from
  the hypervisor. That list is the only way a site is tied to a machine; never any other.
- Regenerate `scripts/vm-specs.json`:
  `nix eval --raw "path:$PWD#lib.vmSpecsJson" | jq -S . > scripts/vm-specs.json`.
- `allod check .`, `allod change record`, `allod change submit` with an R0 body.

After the merge, `pull-all` on BOTH the hypervisor and the VM: `view` resolves the id on
the hypervisor, then runs `serve <id>` inside the VM, which reads the VM's own inventory
checkout; a stale VM copy answers `unknown site: <id> has no entry in
<inventory checkout>/scripts/repositories.json`, a path that reads the same on either
machine. Then stop any preview started from the directory (its unit is
`allod-preview-work-<path>`) and start it by id (`allod-preview-<id>`), the unit `view`
reuses; two cannot share the port.

`view` printing the address and then `bind [127.0.0.1]:<n>: Address already in use` is a
stale forward on the hypervisor from an earlier hand-made tunnel, held by the shared ssh
master: `ssh -O cancel -L <n>:127.0.0.1:<n> <vm>` releases it; never kill the master.

Until the PR merges, `allod site serve --port <n>` from the clone plus
`ssh -N -L <n>:127.0.0.1:<n> <vm>` on the hypervisor is the fallback.

## 4. Serve and look

On the VM, from the clone: `allod site serve --port <n>`. The registry it reads is
`$INVENTORY/scripts/repositories.json` when `INVENTORY` is set and the `allod/inventory`
checkout under `~/work` otherwise; a shell opened before the VM's last rebuild may lack
the variable, so set it by hand there. The unit keeps running under systemd; `--stop`
ends it; the server watches the files, so edits show on reload. From the hypervisor
after the merge: `allod site view <id>`.

No dev VM has a browser. To check a layout, fetch chromium for the session with
`nix build --no-link --print-out-paths --inputs-from . nixpkgs#chromium` (several
hundred megabytes; check `df -h /` first) and screenshot
`http://127.0.0.1:<n>/#<slide>` at 1920x1080:
`<path>/bin/chromium --headless --no-sandbox --disable-gpu --hide-scrollbars
--user-data-dir=<scratch> --virtual-time-budget=3000 --screenshot=<file>
--window-size=1920,1080 <url>`. Without the time budget a fade-in animation is caught
half way and the page looks dim.

## 5. Live editing

The owner reviews on their screen while asking for changes: commit each edit as it lands
and push, one commit per request; batching has been corrected before. Fetch before
handing over any relay line for a public repo the owner may already have landed.
