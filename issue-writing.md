# Issue Writing

## User Story

Open every issue body with a one- or two-sentence plain-English user story before any technical detail: the operator-facing outcome ("So that I can ..."), readable by someone who has not seen the code.

When a story spans multiple issues or repos, give it one home in `allod/strategy` and link every child to it:
- Big arc (multi-PR): the `strategy/dev-plans/<slug>.md` Goal is the story; each issue links to the plan.
- Smaller multi-repo arc: one tracking issue in `allod/strategy` stating the story, with children as a cross-repo task list (`- [ ] allod/<repo>#N`).

Each child leads with the same one-line story and `Part of allod/strategy#N`. Keep the canonical story in the single home so it does not drift.

## Implementation Issues

Write each issue as a request for one artifact. The title names the first repo-scoped change; the body opens with the user story (above), then the artifact.

Separate facts from asks:
- Context: behavior, versions, logs, suspected causes.
- Scope: files, inputs, defaults, or scripts to change.
- Validation: rebuilds, commands, interactive checks, product verification, or manual gates.

Define boundaries, including out-of-scope overrides, downstream repos, host operations, secrets, rebuilds, and manual tests.

If several paths exist, name the default and leave alternatives as follow-ups unless it fails.

For package/version issues, specify lock bump, override, downstream default change, or tracking issue.

Mark exploratory work explicitly; do not call it implementation-ready until the artifact and stopping point are clear.
