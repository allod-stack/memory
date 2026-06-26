# Security Practices

## Token Handling

- No token-print, token-export, or token-display commands.
- No token argv or token-path inputs by default.
- Candidate tokens use stdin: `producer | tool verify`.
- API auth must keep tokens out of child argv. For `curl`, use `--config -` or a private file descriptor, not `-H "...$TOKEN..."`.
- If another tool needs auth, build a narrow credential helper, not a generic token dump.
