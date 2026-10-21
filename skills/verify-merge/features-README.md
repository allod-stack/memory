# <Project> feature map

Behavior-level inventory of <project>, one file per area. Agents use it to
decide what to drive for a change and what evidence counts; humans use it
as the regression checklist. The conventions are `testing.md` (Feature Map)
in allod/memory; this file carries what is specific to this repository.

Copy this template to `verification/features/README.md`, replace every
angle-bracket placeholder, and delete this paragraph.

## Driver

The commands a verifier runs, each one a real command in this repository
(a justfile target, a script, a cargo alias), not a description:

| Step | Command | What it proves |
| --- | --- | --- |
| launch | `<command>` | the product is running from this checkout |
| health check | `<command>` | the right build and instance, fresh state; a stale build is reported, not driven |
| drive | `<command> <entry point>` | one user-reachable entry point exercised end to end |
| inspect | `<command>` | the observable state after a path ran: a query, a log line, a file |
| clean up | `<command>` | nothing from this run leaks into the next |

## Baseline preconditions

- Checks that must pass before driving anything: `<command>`.
- <Fresh state, seed data, test credentials, ports, and where each comes from.>
- <What is manual or unreachable from a dev VM, and why.>

## Proof standard

For a change, the touched feature files define the coverage set. Each
entry point in them is driven through its success, cancel, error, empty and
persistence paths where the change can affect them, and every row of proof
names the expected observable result before the observed one. A path that
cannot be reached is reported as a gap with what blocks it, never as a pass.
A mock counts only behind the same boundary an unavailable external
dependency would use.

## Sweep order

For a broad regression, walk the files top to bottom:

- [<area>](<area>.md): <one line of what it covers>.
- [<area>](<area>.md): <one line>.

## Entry contract

Every feature file uses the same four headings, in this order:

1. `What exists` — the behaviors, one line each, with the variants that
   change them.
2. `How a user reaches it` — the entry points: command, endpoint, UI path,
   timer, with preconditions.
3. `How to drive it` — the driver invocation per entry point, and the
   expected observable result of each path.
4. `What usually lies` — the false passes: a cached result, a default that
   masks the error path, a log line that prints before the write commits.

Map drift is fixed in the same PR as the behavior change. A PR that removes
or narrows an entry here is never merged below R2.
