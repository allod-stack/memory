---
name: age-encrypt-secret
description: Turn a secret into an age ciphertext without the secret ever appearing on the terminal, in a shell's argv or history, or in a temp file; also the paste-safe command shape for anything a human will run by hand. Use whenever a credential must be age-encrypted to a recipient set, and read before relaying any multi-line command a human pastes into a terminal.
---

# age-encrypt-secret

One question decides how to encrypt a secret: **would the secret survive the command?** A key echoed to the terminal, passed as an argument, or written to a temp file is a copy of the credential with a longer life than the moment you meant it for. This ritual keeps the secret in exactly two places: the human's hands while typing, and the `.age` ciphertext afterwards.

## The ritual

```bash
read -rs -p 'Paste the secret, then press Enter: ' SECRET
SECRET=${SECRET//[[:space:]]/}
[ -n "$SECRET" ] || { echo 'nothing pasted; aborting'; exit 1; }
printf '%s' "$SECRET" | age -R <(printf '%s\n' '<age-recipient-1>' '<age-recipient-2>') -o path/to/output.age && echo encrypted
unset SECRET
age -d -i ~/.ssh/<identity> path/to/output.age | grep -q '^<expected-prefix>' && echo ok || echo 'decrypt check failed'
```

What each line does, and why it is shaped that way:

- `read -rs -p …` — `-p` prints the prompt so the reader knows input is awaited; `-s` echoes nothing while typing. This is the only line that handles the secret.
- `SECRET=${SECRET//[[:space:]]/}` — strips whatever a paste drags in (trailing newline, CRLF, stray spaces). Do this for any single-token secret; tailscale auth keys and similar tokens contain no internal whitespace, so it is lossless.
- `printf '%s' "$SECRET"` into a pipe — the value travels as stdin, never as a command argument, so it cannot appear in `ps` or shell history.
- `-R <(printf '%s\n' …)` — recipients ride in process substitution (an open fd, not a file on disk); no temp file means no cleanup step and nothing of the plaintext touches disk at all.
- `unset SECRET` — drop the value from the shell as soon as the pipe closed.
- The verification line pipes the decrypted value into `grep -q`: exit status only, nothing printed. Never replace it with a bare `age -d …` that echoes the secret, and never echo the plaintext back for confirmation.

## Recipients

Recipient keys are public and come from the deployment's key registry (`machine-host-keys.json`, `nexus-host-key.json`, or their fork equivalents) — never from memory of a command. A changed recipient set means re-encrypting the ciphertext: a machine missing from the set can no longer decrypt, while the ciphertext keeps working for those who remain.

## Paste-safe relay shape

Anything handed to a human to run by hand will be mangled by the copy step, and the mangling is silent:

- Rich-TUI selection gains a two-space gutter and can hard-wrap INSIDE a token or at a line continuation. Backslash continuations are the classic casualty: the wrap turns `x \` plus the next line into two commands, or drops the backslash entirely — the symptom is `age: error: missing recipients` when the recipients lived on the dropped line.
- Therefore: **no backslash line continuations in relayed commands; make every relay line valid shell on its own** — one complete command per line.
- Select with the tool's raw, copy-friendly mode (e.g. `/raw on` in pi's rich TUI) rather than by mouse.
- If a command still fails with "command not found" on a flag or "missing recipients", the copy lost its shape — re-derive the lines from the source file instead of hand-repairing the mangled text.

## When a script is the better shape

If the command is long, or the reader will paste it into a captured session, write a temporary script instead and use the hash-checked transfer: show the human the readable script, have them verify its hash after transfer, then execute the file. Never relay content the human cannot inspect, and never ask them to run an opaque encoded blob.

## Boundaries

This ritual covers the plaintext's journey into the ciphertext. It does not authenticate the recipient keys, store the plaintext anywhere, or decide rotation policy — that is the credential's own lifecycle in the secrets repo.
