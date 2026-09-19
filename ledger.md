# Ledger

Sightings waiting for corroboration. Not read at session start. One line per sighting, appended at the end of a session or by the `memory-backpass` skill. A rule enters `memory.md` once two distinct sessions have sighted it (`memory.md`, Memory File Hygiene). A sighting is removed once the rule it argues for lands in a topic file or the index, or after ninety days.

Format, one line each:

- <kind>: <the rule, one imperative sentence> | <harness> <session id> | "<verbatim quote from the transcript that shows the problem>"

Kinds: `gap` for a mistake no current line covers, `ignored` for a line that was not followed (name the line by its opening words), `harm` for a line that was followed and caused damage. Only `harm` sightings argue for removing a line.

- gap: After adding a refusal fixture, negative-control it before trusting a green run: run the clause it targets out of the predicate and confirm the test goes red, since a refusal case can pass for an unrelated reason. | claude 65572a28-14d6-49a2-a981-393d7cbcfc4a | "the new fixture wrote the carriage return on the URL line rather than on a second line, so 'sabotage.carriage-return' passed on the URL regexp and never exercised the blank-line class it was named for"
- gap: After merging master into a PR branch, diff the result against master for the same fact bound twice; a merge keeps both sides' bindings and the checks still pass. | claude 4699f436-8a23-4c3d-ae78-72c13e1d00be | "master (PR 30) had introduced its own let binding forgejoTokenGroups = validateCredentialRegistry (...) and the merge kept it beside ours, so flake.nix now validates the registry twice"
