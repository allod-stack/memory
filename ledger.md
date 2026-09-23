# Ledger

Sightings waiting for corroboration. Not read at session start. One line per sighting, appended at the end of a session or by the `memory-backpass` skill. A rule enters `memory.md` once two distinct sessions have sighted it (`memory.md`, Memory File Hygiene). A sighting is removed once the rule it argues for lands in a topic file or the index, or after ninety days.

Format, one line each:

- <kind>: <the rule, one imperative sentence> | <harness> <session id> | "<verbatim quote from the transcript that shows the problem>"

Kinds: `gap` for a mistake no current line covers, `ignored` for a line that was not followed (name the line by its opening words), `harm` for a line that was followed and caused damage. Only `harm` sightings argue for removing a line.

- gap: After adding a refusal fixture, negative-control it before trusting a green run: run the clause it targets out of the predicate and confirm the test goes red, since a refusal case can pass for an unrelated reason. | claude 65572a28-14d6-49a2-a981-393d7cbcfc4a | "the new fixture wrote the carriage return on the URL line rather than on a second line, so 'sabotage.carriage-return' passed on the URL regexp and never exercised the blank-line class it was named for"
- gap: To diff two versions of a Nix file with comments and formatting stripped, parse both with `nix-instantiate --parse` from the same directory (stage the base copy beside the file), because the parser resolves relative path literals against the parsed file's directory and a copy parsed from a temp dir reads as changed. | claude fe4a8e76-9e48-4d7d-a8e1-ca05e0e0ceef | "the gate writes the master copy to $(mktemp -d)/old.nix and parses it there; nix-instantiate --parse resolves a path literal against the parsed file's directory, so import ./allod-package.nix prints as /tmp/tmp.HfWFfhOD0A/allod-package.nix on the master side"
- gap: Before deleting a file an issue says exists only to satisfy one check, grep every reader of it in the repo and its fixtures; the claim is the author's hypothesis, and a fixture file often feeds a predicate and a floor besides the check named. | claude 7616738f-4069-46d0-85da-90206e21c5df | "drop the table from checks/fork-fixture/secrets/, which exists only to satisfy this check" — the fixture's own isCredentialStoreUrlTemplate read credentialStoreUrl.line and credential-profiles floored its vectors
- gap: Re-measure a memory or timing figure quoted from a topic file before letting it constrain how work is scheduled; the archetypes "peaks around 5.7 GiB" note predated the gate split and the real per-process peak was 1.7 GiB. | claude 65544cc3-0545-4ee4-a93a-41ff09331efa | "This box has 7 GiB and the check suite peaks near 5.7 GiB, so two suites at once gets one OOM-killed"
