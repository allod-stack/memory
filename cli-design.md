# CLI Design

A CLI is a human-facing interface. Apply ordinary UX judgment to it, and settle on the interface a person would want to use before weighing what it costs to build.

- A plural flag name takes multiple values. `--files a b c` works, repeating the flag accumulates, and the two forms combine.
- A following option ends a variadic list, so flag order stays free; `--` ends option parsing so a value beginning with `-` stays reachable.
- A list that consumed nothing is an error, never a silent fall back to a broader default.

When usage text and behavior disagree, the default is to fix the behavior. Editing documentation to match a defective implementation documents the bug, and needs a real reason beyond the smaller diff.

`architecture.md` principle 15's minimalism governs what gets built, not how poor an interface may be — usability sits in that same priority list. Diff size is not an argument for a worse interface.
