# Pi Adapter

At conversation start, before responding to the first user message, read `../../memory.md` relative to this adapter file — the repository root. If that path does not resolve, locate the file before continuing; proceeding with memory unread is a defect.

The pointer above is repeated in each harness's adapter deliberately: a shared include would trade it for an extra file read at the start of every session. Do not extract it.
