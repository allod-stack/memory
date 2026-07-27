# Agent Behavior

## Scope Discipline

For issue work, identify the smallest repo-scoped artifact that advances the request, and make that change before exploring broader end-to-end fixes.

When an issue names multiple possible paths, separate them into immediate change, validation, manual test, and follow-up. Do not choose the most ambitious path by default.

Do not run expensive builds, package rebuilds, VM rebuilds, broad searches, local installs, or environment mutations to validate an alternate path unless the user asked for that path or the repo change cannot be selected without it.

For package/version tasks, prefer this order: bump the existing lock or package reference; do lightweight metadata checks if needed; add overrides only after the simple path is insufficient and accepted; build only when required by acceptance tests or explicitly requested.

Treat a user interruption as a hard scope signal: stop the current path, avoid adjacent exploration, and resume from the narrowest reading of the latest instruction.

Read user-provided paths and names literally. A leading dot (e.g. `allod/.profile`) is a real repo — the Forgejo org-profile repo — not a typo for a similarly named one; verify the exact path before acting on a look-alike.

Do not assume every machine is running. VMs are disposable and started on demand; unreachable is normal.

Several agents may run in one VM — supported, not accidental. The workspace is shared, so a dirty tree or an unexpected branch may be another agent's. Leave work in progress alone and pick something else.

## Crossing the Public/Private Boundary

An agent holding private material cannot write public org repos; pushes are refused server-side. That is the boundary working — do not look for another transport or open the change from a fork.

Hand off a public code change as a real commit on an `agent/<description>` branch in the public repo's own checkout, left unpushed for the human to relay. Never embed it as a patch in a notes or plan document; only sanitized issue and plan prose belongs there. Prepare it in the public checkout — a fork drifts, and a change built against the fork may not apply.
