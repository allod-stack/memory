# age / agenix Workflows

## Safe Secret Input: No Tempfile

Secret input must never touch the filesystem. Pipe directly from `read` to `age`:

```bash
# Single recipient:
IFS= read -r -s -p "Paste secret: " tok && echo
(umask 077; printf '%s' "$tok" | age -e -R ~/.ssh/host.pub -o secrets/<name>.age)
unset tok

# Multiple recipients:
IFS= read -r -s -p "Paste secret: " tok && echo
(umask 077; printf '%s' "$tok" | age -e -r "ssh-ed25519 <key1>" -r "ssh-ed25519 <key2>" -o secrets/<name>.age)
unset tok
```

- `read -r -s` - no echo, no history
- `printf '%s'` - no trailing newline
- pipe to `age` - secret never hits disk
- `umask 077` subshell - output file gets 600
- recipients must match `secrets.nix` for agenix to decrypt on the target machine

## Running agenix from secrets

When the secrets repo has no direct agenix input, use the VM flake app that exposes it:

```bash
cd <workspace>/allod/secrets
nix run <workspace>/allod/vm#agenix -- -e secrets/<name>.age -i ~/.ssh/host
```

agenix reads `secrets.nix` from the current working directory, so running the VM app from the secrets directory works correctly.

## Recipient Keys

Recipient public keys are declared in `secrets/secrets.nix`.
The host SSH key can double as the age identity for decryption when configured that way.

## Common Agent Mistake: Wrong Host Key Path

Host identity key paths are deployment-specific. Prefer the configured identity path for the target machine instead of guessing from `/etc/ssh`.
