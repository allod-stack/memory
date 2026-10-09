# Retired

Index lines the `memory-prune` skill removed, newest block first. Not read at session start.

This is the restore source. To put a line back, copy it verbatim out of its block and into the section the block names, or revert the commit the block names. A line listed here is not re-added by any pass without new evidence (`memory.md`, Memory File Hygiene), and `memory-prune` never retires an identity that already has a block here — a line that came back after being retired is a decision to look at, not one to repeat.

Format, one block each:

    ## I-<hash> <oldest trace date>..<newest trace date>

    <the removed line, verbatim>

    <section it came from> | <class> | steered <n> ignored <n> harm <n> of <m> scored sessions | <branch that removed it>

`I-<hash>` is the first eight hex characters of the SHA-256 of the line's whitespace-folded text, the identity `memory-prune` scores it under. The two dates bound the window of sessions that measured it; `<class>` is `unused` when no session in that window drew any sighting against the line, or `inert` when the only sightings were of it being ignored, across two consecutive windows.
