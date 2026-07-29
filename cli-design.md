# CLI Design

A CLI is a human-facing interface. Apply ordinary UX judgment to it, and settle on the interface a person would want to use before weighing what it costs to build.

Copy `gh` wherever it has an established shape for the case at hand — flag names, short aliases, and value-list handling follow it rather than a local invention.

## Multi-value flags

Two shapes are in use and the rule above them is unsettled; `allod/tools#131` decides it.

- `forge`'s label flags (`-l|--label`, `--add-label`, `--remove-label`, `--set`) take comma-separated values, and repeating the flag accumulates. This is `gh`'s shape, built on pflag's string-slice type.
- `allod change record --files` is space-variadic: `--files a b`, `--files a --files b`, and `-f a b --files c` all name the same set. A comma is legal in a filename, which is why `gh` takes path lists as variadic positionals rather than comma-separated flags.

Either way:

- A following option ends a variadic list, so flag order stays free; `--` ends option parsing so a value beginning with `-` stays reachable.
- A list that consumed nothing is an error, never a silent fall back to a broader default.

## Usage text

When usage text and behavior disagree, the default is to fix the behavior. Editing documentation to match a defective implementation documents the bug, and needs a real reason beyond the smaller diff.

`architecture.md` principle 15's minimalism governs what gets built, not how poor an interface may be — usability sits in that same priority list. Diff size is not an argument for a worse interface.
