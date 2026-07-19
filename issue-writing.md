# Issue Writing

## Shape

Every issue body follows one shape. Lead with utility, then goals, then detail — never bury the point under preamble.

1. **One plain opening sentence** stating what the change achieves, understandable by someone who has not seen the code. State the utility directly. Do NOT open with a "User story" / "So that I can ..." preamble, and do NOT prepend PM-arc or board-plan metadata lines — that boilerplate is exactly what makes issues unreadable. When the change brings things up to an existing better standard, say so plainly: "Bring X, Y, Z up to the same standard as W."
2. **A short `Primary goals:` bullet list** — 3-6 skimmable bullets, each a bolded lead-in plus one concrete outcome. Not paragraphs.
3. **One or more `###` detail sections** for the technical specifics: current state, affected files with `path:line` receipts, mechanisms, and defaults.
4. **A `### Scope` section** whenever scope can bleed: what is in, and what is explicitly tracked elsewhere.

Prose as single long lines; break only at paragraphs, list items, and code blocks. Reference other issues as `owner/repo#N` with the issue/PR word in prose.

Skeleton:

    <one plain sentence: what this achieves, or what it brings up to standard>

    Primary goals:

    - **<lead-in>** — <concrete outcome>
    - **<lead-in>** — <concrete outcome>

    ### Current state and specifics

    <current behavior, path:line receipts, mechanisms, defaults>

    ### Scope

    <in scope here; what is handled by separate work>

## Facts vs asks

Within the detail sections keep facts and asks separate:
- Context: behavior, versions, logs, suspected causes.
- Scope: files, inputs, defaults, or scripts to change.
- Validation: rebuilds, commands, interactive checks, product verification, or manual gates.

Name the default when several paths exist; leave alternatives as follow-ups unless the default fails. For package/version issues, specify lock bump, override, downstream default change, or tracking issue. Mark exploratory work explicitly; do not call it implementation-ready until the artifact and stopping point are clear.

## Cross-repo arcs

When one effort spans multiple issues or repos, keep its canonical goal statement in a single home in `allod/strategy` — a `strategy/dev-plans/<slug>.md` Goal for a multi-PR arc, or one tracking issue for a smaller multi-repo arc — and link each child with `Part of allod/strategy#N` plus a cross-repo task list (`- [ ] allod/<repo>#N`). Each child still opens with its own plain utility sentence; the shared home only prevents the goal from drifting.
