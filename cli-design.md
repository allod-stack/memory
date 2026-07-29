# CLI Design

A CLI is a human-facing interface. Apply ordinary UX judgment to it, and settle on the interface a person would want to use before weighing what it costs to build.

Copy `gh` wherever it has an established shape for the case at hand — flag names, short aliases, and value-list handling follow it rather than a local invention.

## Multi-value flags

- Value lists take comma-separated values, and repeating the flag accumulates: `forge --label a,b` and `forge --label a --label b` name the same set. This is `gh`'s shape, and `forge` already follows it — `-l|--label`, `--add-label`, `--remove-label`, `--set`. Space-separated values after such a flag are an error, not a second spelling.
- Path lists are the exception, because a comma is legal in a filename. `gh` takes them as variadic positionals; `allod change record --files` takes them space-separated, where `--files a b`, `--files a --files b`, and `-f a b --files c` all name the same set.

Either way:

- A following option ends a variadic list, so flag order stays free; `--` ends option parsing so a value beginning with `-` stays reachable.
- A list that consumed nothing is an error, never a silent fall back to a broader default.

## Usage text

When usage text and behavior disagree, the default is to fix the behavior. Editing documentation to match a defective implementation documents the bug, and needs a real reason beyond the smaller diff.

`architecture.md` principle 15's minimalism governs what gets built, not how poor an interface may be — usability sits in that same priority list. Diff size is not an argument for a worse interface.
