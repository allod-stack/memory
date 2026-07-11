# Issue Writing

## Implementation Issues

Write each issue as a request for one artifact. The title and opening sentence should name the first repo-scoped change.

Separate facts from asks:
- Context: behavior, versions, logs, suspected causes.
- Scope: files, inputs, defaults, or scripts to change.
- Validation: rebuilds, commands, interactive checks, product verification, or manual gates.

Define boundaries, including out-of-scope overrides, downstream repos, host operations, secrets, rebuilds, and manual tests.

If several paths exist, name the default and leave alternatives as follow-ups unless it fails.

For package/version issues, specify lock bump, override, downstream default change, or tracking issue.

Mark exploratory work explicitly; do not call it implementation-ready until the artifact and stopping point are clear.
