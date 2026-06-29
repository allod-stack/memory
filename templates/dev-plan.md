# [Plan Name]

## Tracking Issue

[Forge issue URL or number. If the work spans multiple PRs, state which PR carries the closing keyword.]

## Goal

[One sentence describing the user-visible outcome.]

## Scope

In scope:

- [Files, services, modules, repos, or behaviors this plan changes.]

Out of scope:

- [Nearby work that this plan intentionally does not change.]

## Risk Assessment

Residual risk: R[0-4] [Docs/metadata | Low | Medium | High | Critical]

Why:

- [Blast radius and intended behavior change.]
- [Validation evidence that lowers or fails to lower risk.]
- [Rollback confidence.]

Human scrutiny:

- [What the human should look at first: diff shape, generated artifacts, lifecycle behavior, secrets/privacy boundaries, cross-repo ordering, or validation output.]

For multi-PR plans:

| PR or milestone | Risk | Reason | Human scrutiny |
|---|---|---|---|
| [name] | R[0-4] | [why] | [what to inspect] |

## Interface Contracts

[Function signatures, CLI flags, file formats, Nix attributes, API shapes, or generated artifacts the implementation must match. If none apply, state that explicitly.]

## Agent Gates

[Actions the agent cannot perform or should not perform without human input, what the human must do, and what it blocks. Omit this section only if no gates apply.]

## Acceptance Tests

```sh
# Concrete commands the agent can run locally before declaring the task complete.
```

## Rollback Plan

[How to revert the change if verification fails, including partial states when relevant.]
