# Agent Behavior

## Scope Discipline

For issue work, identify the smallest repo-scoped artifact that advances the request, and make that change before exploring broader end-to-end fixes.

When an issue names multiple possible paths, separate them into immediate change, validation, manual test, and follow-up. Do not choose the most ambitious path by default.

Do not run expensive builds, package rebuilds, VM rebuilds, broad searches, local installs, or environment mutations to validate an alternate path unless the user asked for that path or the repo change cannot be selected without it.

For package/version tasks, prefer this order: bump the existing lock or package reference; do lightweight metadata checks if needed; add overrides only after the simple path is insufficient and accepted; build only when required by acceptance tests or explicitly requested.

Treat a user interruption as a hard scope signal: stop the current path, avoid adjacent exploration, and resume from the narrowest reading of the latest instruction.

Read user-provided paths and names literally. A leading dot (e.g. `allod/.profile`) is a real repo — the Forgejo org-profile repo — not a typo for a similarly named one; verify the exact path before acting on a look-alike.

Do not assume every machine in the fleet is running. VMs are disposable and started on demand, so a machine being unreachable is normal and not evidence that anything is broken.

Several agents may run in one VM at once; that is a supported configuration, not an accident. Treat the workspace as shared: another agent may hold a checkout, be mid-change in a repo you are only reading, or be working the same issue you just picked. A dirty tree or an unexpected branch is not necessarily yours. When you find another agent's work in progress, leave it alone and pick different work rather than reconciling on its behalf.

## Crossing the Public/Private Boundary

An agent holding private material cannot write public org repos: pushes are refused server-side, and that is the boundary working, not a misconfiguration to route around. Do not look for another transport, and do not open the change from a fork.

Hand a public code change off as a real commit — on an `agent/<description>` branch in the public repo's own checkout, left unpushed for the human to relay. Never embed a public code change as a patch inside a notes or plan document: only sanitized issue and plan prose belongs in notes, and a doc-embedded diff is not reviewable or appliable as code.

Diff against the right base. A public repo and a private fork of it drift, so a change prepared against the fork may not apply to the public repo at all; prepare it in the public checkout.
