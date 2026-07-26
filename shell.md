# Shell Gotchas

These bite hardest in assertion and scrub scripts, where the failure mode is a check that silently always passes. A guard that cannot be shown to fail on sabotaged input does not count (`architecture.md` principle 11), so write each of these the loud way.

## `set -e` exempts inverted and non-final commands

Under `set -e`, `! cmd` never aborts the script — bash exempts `!`-inverted commands — so a `! rg <forbidden-token>` scrub assertion is a silent no-op. Write it as a conditional instead:

```bash
if rg <forbidden-token> .; then exit 1; fi
```

The same family: in `a && b && c` only the final command's failure aborts, because non-final members of an `&&` list are exempt too. An assertion chain that ends in `echo` can never fail. Put one assertion per line.

## `jq -r` prints the string `null` for a missing path

A missing path yields the four characters `null` and exit status 0, which defeats `[ -n "$value" ]`. Append `// empty` to the filter so a missing path produces an empty string:

```bash
value=$(jq -r '.some.path // empty' file.json)
```

## Double-escaped regex metacharacters in single quotes

Inside single quotes the shell does no backslash processing, so `'\\+'` reaches the regex engine as an escaped backslash followed by `+` — it matches literal backslashes, not one-or-more. Single-quote the pattern and escape it once: `'\+'`.
