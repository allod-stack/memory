# Dev Plan Guidelines

Every dev plan must include these sections. Template: `allod/memory/templates/dev-plan.md`.

1. **Tracking Issue** - Forge issue URL or number for the work. If multiple PRs are expected, state which PR should carry the closing keyword.

2. **Goal** - One sentence describing the user-visible outcome.

3. **Scope** - Which files, services, or modules are in scope and which are explicitly out of scope.

4. **Risk Assessment** - The residual risk level after the planned validation passes, why that level is appropriate, and what human scrutiny is most useful.

5. **Interface Contracts** - Any function signatures or API shapes the agent must match. If none apply, state that explicitly.

6. **Agent Gates** - Actions the agent cannot perform, such as creating repos, deploying, granting permissions, running host-only commands, or changing secrets. State what the human must do and what it blocks. Omit only if none apply.

7. **Acceptance Tests** - Tests the agent can run locally before declaring the task complete. Must be concrete and executable. For NixOS, Home Manager, provisioning, systemd, or wrapper-script changes, include checks against generated lifecycle artifacts and expected absent-resource paths, not only source evaluation or happy-path builds.

8. **Rollback Plan** - How to revert the change if verification fails.

## Risk Assessment

Use one residual risk score for human triage. Residual risk means how much scrutiny is still useful after the plan's validation passes. Prefer this over inherent risk in normal plans; mention inherent risk only when a risky area is substantially reduced by tests, rollout order, or rollback.

Risk levels:

- **R0 Docs/metadata** - No runtime behavior, generated behavior, operational behavior, secrets, or state changes.
- **R1 Low** - Refactor-only or very localized behavior; caller-level tests or equivalent checks give high confidence; rollback is a straight revert; no secrets, host-only commands, provisioning, activation, persistent state, or cross-repo ordering.
- **R2 Medium** - Localized behavior change or refactor with some integration uncertainty; main paths are tested; rollback is straightforward.
- **R3 High** - Cross-repo interfaces or sequencing, generated lifecycle behavior, systemd units, wrappers, provisioning, auth, secrets, privacy/security boundaries, persistent state, dependency/toolchain changes, or broad refactors without caller-level coverage.
- **R4 Critical** - Destructive or hard-to-rollback operations, first-boot/provisioning paths that can strand a machine, security/privacy boundary changes where a mistake could expose private material, irreversible data/schema changes, or validation that depends on human-only infrastructure.

Risk signals raise scrutiny; they do not automatically block implementation. Add an Agent Gate only when the agent cannot perform the action, lacks the needed environment, or the human must make a real decision. High residual risk should usually lead to clearer acceptance tests, rollback steps, generated-artifact inspection, or a review pass before it leads to ceremony.

For multi-PR plans, assign risk per PR or milestone. The score can change during implementation if the diff, validation, or rollback story changes.

## Issue and PR Linkage

- If no tracking issue exists, create one before implementation PRs are opened.
- Add the issue URL or number to the dev plan before implementation starts.
- Reference the issue from every implementation PR.
- For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and `Closes owner/repo#N` only on the final integration PR.
- If the final PR changes during implementation, update the dev plan and PR bodies.
- Leave archived strategy dev plans alone unless asked.

## Plan Review

Iterative review template: `allod/memory/templates/plan-review-prompt.md`.

Review prompts live in the same repository as the dev plan they review.
See `allod.md` for `allod/strategy` subdirs.

### Standing Focus Areas

These six lenses apply to every review pass as defaults. Specific focus areas from previous passes replace or supplement them when the prompt is updated between passes.

Verify prereqs, and delete each one once met: replace an interface prereq with the contract it now provides, and drop a satisfied sequencing gate (e.g. "lands after branch X merges") outright. Do not keep met preconditions as historical notes — a plan or review prompt is not a log of legacy implementations.

1. **Internal consistency** - Do Interface Contracts, PR descriptions, and Acceptance Tests agree?
2. **Operational sequencing** - Can someone execute the plan cold without getting stuck?
3. **Risk calibration** - Does the residual risk level match the blast radius, rollback path, and validation evidence?
4. **Acceptance test coverage** - Do tests exercise every contract rule? Do all tests trace to a requirement?
5. **Rollback fidelity** - Does the rollback plan undo what the implementation does, including partial states?
6. **Generated lifecycle behavior** - Do generated activation scripts, systemd units, wrappers, provisioning phases, rebuild paths, and missing optional resources behave correctly?

### Agent Rotation

Rotate agents between review passes to get different perspectives. Each agent has different blind spots; rotating broadens coverage. The focus areas section and commit history provide continuity so a fresh agent can pick up context without duplicating prior work.

Track fix stability per model. In the review prompt's focus-area updates, record how each model's fixes held up: a fix that survived N later passes is stable; a fix that needed immediate re-fixing in the next pass is not. Review passes are launched manually, one model per pass, so the following are recommendations the finishing pass records for whoever starts the next pass — the agent cannot select the next model itself:

- Prefer the model with the best fix-stability record for verification passes (the scoped diff review of a structural fix).
- Drop a model from the rotation after repeated same-feature regressions.
