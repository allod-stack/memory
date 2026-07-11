# Issue Writing

## Implementation Issues

Write implementation issues as a clear request for one artifact, not as a bundle of investigation notes. The title and first sentence should name the repo-scoped change an agent should make first.

Separate context from instructions. Observations, current versions, logs, or suspected causes belong under context. Required file changes belong under scope. Rebuilds, interactive checks, host commands, and product verification belong under validation or manual gates.

If there are multiple possible paths, name the default path and mark alternatives as follow-ups unless the first path fails. Do not leave the agent to infer whether to bump a lock, add an override, rebuild a VM, and update downstream defaults in one pass.

Use explicit scope boundaries:
- In scope: the files, inputs, defaults, or scripts expected to change.
- Out of scope: overrides, downstream repos, host operations, secrets, rebuilds, or manual tests that should not happen in the first PR.

For package/version issues, state whether the desired PR is a lock bump, a package override, a downstream default change, or only a tracking issue. Put manual verification such as `codex --version`, model pickers, or VM rebuilds in a separate validation section.

When the issue is intentionally exploratory, say so. Do not label it as implementation-ready until the first artifact and stopping point are explicit.
