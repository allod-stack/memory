# Residual Risk

One residual risk score, R0–R4, decides how much scrutiny a change still deserves after its validation passes. A dev plan carries it per plan or per PR (`dev-plans.md`); a PR earns its review pass by it (`memory.md`, PR Workflow).

Use one residual risk score for human triage. Residual risk means how much scrutiny is still useful after the plan's validation passes. Prefer this over inherent risk in normal plans; mention inherent risk only when a risky area is substantially reduced by tests, rollout order, or rollback.

Risk levels:

- **R0 Docs/metadata** - No runtime behavior, generated behavior, operational behavior, secrets, or state changes.
- **R1 Low** - Refactor-only or very localized behavior; caller-level tests or equivalent checks give high confidence; rollback is a straight revert; no secrets, host-only commands, provisioning, activation, persistent state, or cross-repo ordering.
- **R2 Medium** - Localized behavior change or refactor with some integration uncertainty; main paths are tested; rollback is straightforward.
- **R3 High** - Cross-repo interfaces or sequencing, generated lifecycle behavior, systemd units, wrappers, provisioning, auth, secrets, privacy/security boundaries, persistent state, dependency/toolchain changes, or broad refactors without caller-level coverage.
- **R4 Critical** - Hard-to-rollback operations where the worst credible failure can broadly, implicitly, or opaquely affect unique, hard-to-reconstruct, deployed, security-sensitive, or infrastructure state; first-boot/provisioning paths that can strand a machine; security/privacy boundary changes where a mistake could expose private material; irreversible data/schema changes; or validation that depends on human-only infrastructure.

Risk signals raise scrutiny; they do not automatically block implementation. Add an Agent Gate only when the agent cannot perform the action, lacks the needed environment, or the human must make a real decision. High residual risk should usually lead to clearer acceptance tests, rollback steps, generated-artifact inspection, or a review pass before it leads to ceremony.

Calibrate risk by walking through the worst credible failed run after planned validation passes:

- **Affected state** - Is the state generated, disposable, remote-backed, user-authored, secret, deployed, shared, or unique?
- **Authority of state** - Which copy is authoritative for the workflow: local working state, a remote, declared source files, generated output, deployed state, a backup, or something else? Does the change mutate that authority or only a reconstructable derivative?
- **Blast radius** - Can the failure affect one file, one repo, many local repos, a host, a service, remote state, or other users?
- **Recoverability** - Is recovery a straight revert, a rerun, a reclone, manual repair, backup restore, or impossible from available sources?
- **Propagation** - Is the failure contained in a local workspace, or can it cross into remotes, provisioning, auth, secrets, service availability, or privacy boundaries?
- **Intent and detectability** - Is the risky action explicit and clearly named, or implicit in a default path? Can the operator see what will be affected before or as it happens?
- **Validation limits** - Can the agent exercise the risky path in fixtures or generated artifacts, or does confidence depend on human-only infrastructure?

Choose the lowest level whose description still matches that residual worst case. Destructive commands, broad scope, or scary implementation details raise scrutiny, but the score comes from authority of state, blast radius, recoverability, containment, operator intent, and validation limits rather than from command names alone. A failure is lower risk when it affects only reconstructable derivative state and leaves the authoritative source intact; it is higher risk when it mutates, destroys, or obscures the authoritative source, or when the plan does not establish which copy is authoritative. Unique local user-authored state is a real recoverability concern, but it is not automatically R4 when the risky operation is explicit, local-only, testable in fixtures, and cannot affect remotes, secrets, provisioning, deployed services, or shared infrastructure. Raise to R4 when loss of unique or hard-to-reconstruct state is broad, implicit, hard to detect, crosses a security/infrastructure/shared-state boundary, or lacks a practical rollback or reconstruction path.

## Pass Budget by Risk

Passes are budgeted by the residual risk score above, and they stop at reality:

- **R0/R1**: one pass, checklist depth.
- **R2/R3**: at most two passes, then land and measure. A finding beyond pass two that concerns executable behavior is settled by executing - a boot, a generated artifact, a fixture - not by a third reading.
- **R4**: rotate to convergence (`dev-plans.md`, Agent Rotation).

A pass does not re-review a claim that an available witness can execute (`testing.md`): it demands the execution instead. Repeated passes over unexecuted specification do not converge - each pass finding new defects in never-run behavior is the signal to go run it, not to schedule another pass.
