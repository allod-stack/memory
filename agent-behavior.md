# Agent Behavior

## Scope Discipline

For issue work, identify the smallest repo-scoped artifact that advances the request, and make that change before exploring broader end-to-end fixes.

When an issue names multiple possible paths, separate them into immediate change, validation, manual test, and follow-up. Do not choose the most ambitious path by default.

Do not run expensive builds, package rebuilds, VM rebuilds, broad searches, local installs, or environment mutations to validate an alternate path unless the user asked for that path or the repo change cannot be selected without it.

For package/version tasks, prefer this order: bump the existing lock or package reference; do lightweight metadata checks if needed; add overrides only after the simple path is insufficient and accepted; build only when required by acceptance tests or explicitly requested.

Treat a user interruption as a hard scope signal: stop the current path, avoid adjacent exploration, and resume from the narrowest reading of the latest instruction.
