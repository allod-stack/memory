# age / agenix Workflows

Three of agenix's failure modes exit 0, so a run that looks successful is not evidence that anything happened. They are listed first because each one costs a rebuild or a live credential to discover.

## Three silent no-ops

- **A recipient-only change does nothing under `agenix -e`.** Adding a machine to a `publicKeys` list leaves the plaintext identical, so `-e` prints `<file> wasn't changed, skipping re-encryption.`, exits 0, and the added machine still cannot decrypt (`age: error: no identity matched any of the recipients`) until its next rebuild fails. Recipient changes go through `agenix -r`.
- **Re-encrypting the same value does nothing.** The same comparison and the same exit 0. During a rotation that is the difference between storing the new secret and leaving the old one live.
- **An empty capture still encrypts.** The result is a well-formed ciphertext of an empty string that decrypts cleanly and fails only where it is used. Guard the capture with an `if` block; `return` outside a function does not stop, and `exit` takes an interactive shell with it (`shell.md`).

## A credential lands in two halves

An agent's PR against the secrets repo declares the non-secret half: the `credentials.nix` entry with `rotation_state = "pending"`, the `secrets.nix` recipient line, and the rotation registry group in `forgejo-token-groups.json` (`service` is `forgejo` for a UI token, `none` for anything else). The `credential-inventory` check passes with the `.age` file absent while the entry is pending, and refuses a pending entry that has one or an active entry that lacks one, so the PR is green on its own and can never be merged half-landed. The operator then checks out that branch on the host and runs `allod secret create <name>` with the value on stdin: it encrypts to exactly the recipients `secrets.nix` declares, flips the entry to active, runs the checks, and commits and pushes the branch. `allod secret rekey <name>` re-encrypts after a recipient line changes; rotating to a new value stays with `rotate-token`. The command exists only on the host build (`secret` tag); an agent's half is the PR and the one-line handover, never a plaintext. Details: `docs/allod-secret.md` in `allod/tools`.

## secrets.nix owns the recipient list

`secrets/secrets.nix` maps each `.age` path to its recipient keys, and agenix reads it from the working directory — so every invocation runs from the secrets checkout, and anywhere else stops with `error: path '<cwd>/secrets.nix' does not exist`.

For any path that file governs, let agenix derive the recipients. A hand-written recipient list is a second source of truth that drifts silently: the ciphertext decrypts fine for the machines that were included, while the one left out fails during activation on its next rebuild, far from the change that caused it.

## Safe secret input: no tempfile

agenix takes the plaintext from stdin whenever stdin is not a terminal, so nothing needs an editor:

```bash
printf '%s' "$K" | agenix -e secrets/<name>.age
```

`-i <identity>` is needed only when the file already exists, because agenix decrypts the old value first to see whether anything changed. A leftover file from a failed attempt therefore turns the next attempt into a rotation (`No identity found to decrypt <file>`); when it was never committed, delete it and write it fresh.

agenix stages the cleartext in a `mktemp -d` directory and removes it in an exit trap; `TMPDIR=/run/user/$(id -u)` keeps that staging on tmpfs. Verify without printing the value — `agenix -d <file> -i <identity> | wc -c`, and `grep -ac '^-> ' <file>` counts recipient stanzas, which must match what `secrets.nix` lists.

For a ciphertext `secrets.nix` does not govern, pipe straight into `age` rather than writing a temp file — `read -r -s` (no echo, no history), `printf '%s'` (no trailing newline), and a `umask 077` subshell so the output lands 600:

```bash
IFS= read -r -s -p "Paste secret: " tok && echo
(umask 077; printf '%s' "$tok" | age -e -R ~/.ssh/host.pub -o secrets/<name>.age)
unset tok
```

## Running agenix from secrets

When the secrets repo has no direct agenix input, use the VM flake app that exposes it:

```bash
cd <workspace>/allod/secrets
nix run <workspace>/allod/vm#agenix -- -e secrets/<name>.age -i ~/.ssh/host
```

agenix reads `secrets.nix` from the current working directory, so running the VM app from the secrets directory works correctly.

## Common agent mistake: wrong host key path

Host identity key paths are deployment-specific. Prefer the configured identity path for the target machine instead of guessing from `/etc/ssh`.

## The public secrets repo is a template

Its `machine-host-keys.json` entries and `vm-host-keys/*.age` blobs are synthetic, not the live key store — the registered public key for a machine does not match that machine's real host key. Real key material goes in the private fork only; see `vm-provisioning.md`.
