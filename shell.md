# Shell Gotchas

Each of these turns a check into one that always passes. A guard that cannot be shown to fail on sabotaged input does not count (`architecture.md` principle 11).

## `set -e` exempts inverted and non-final commands

`! cmd` never aborts under `set -e`, so a `! rg <forbidden-token>` scrub assertion is a silent no-op. Use `if rg <forbidden-token> .; then exit 1; fi`.

In `a && b && c` only the final command's failure aborts, so an assertion chain ending in `echo` can never fail. One assertion per line.

## `jq -r` prints `null` for a missing path

A missing path yields the four characters `null` with exit 0, defeating `[ -n "$value" ]`. Append `// empty`.

## Double-escaped metacharacters in single quotes

Single quotes do no backslash processing, so a pattern meant to match a literal `+`, written `'\\+'`, reaches the engine as an escaped backslash followed by the `+` quantifier — one or more literal backslashes, which is not what you were looking for. Escape once: `'\+'`. Applies to `rg` and `grep -E`; in BRE a bare `+` is already literal.
