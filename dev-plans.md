# Dev Plan Guidelines

Every dev plan must include these sections:

1. **Tracking Issue** - Forge issue URL or number for the work. If multiple PRs are expected, state which PR should carry the closing keyword.

2. **Goal** - One sentence describing the user-visible outcome.

3. **Scope** - Which files, services, or modules are in scope and which are explicitly out of scope.

4. **Interface Contracts** - Any function signatures or API shapes the agent must match. If none apply, state that explicitly.

5. **Agent Gates** - Actions the agent cannot perform, such as creating repos, deploying, granting permissions, running host-only commands, or changing secrets. State what the human must do and what it blocks. Omit only if none apply.

6. **Acceptance Tests** - Tests the agent can run locally before declaring the task complete. Must be concrete and executable.

7. **Rollback Plan** - How to revert the change if verification fails.

## Issue and PR Linkage

- If no tracking issue exists, create one before implementation PRs are opened.
- Add the issue URL or number to the dev plan before implementation starts.
- Reference the issue from every implementation PR.
- For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and `Closes owner/repo#N` only on the final integration PR.
- If the final PR changes during implementation, update the dev plan and PR bodies.

## Plan Review

Iterative review template: `templates/plan-review-prompt.md`.

Review prompts live in `allod/strategy/review-plans/`, not alongside dev plans.

### Standing Focus Areas

These four lenses apply to every review pass as defaults. Specific focus areas from previous passes replace or supplement them when the prompt is updated between passes.

Verify prereqs; replace resolved ones with implemented contracts.

1. **Internal consistency** - Do Interface Contracts, PR descriptions, and Acceptance Tests agree?
2. **Operational sequencing** - Can someone execute the plan cold without getting stuck?
3. **Acceptance test coverage** - Do tests exercise every contract rule? Do all tests trace to a requirement?
4. **Rollback fidelity** - Does the rollback plan undo what the implementation does, including partial states?

### Agent Rotation

Rotate agents between review passes to get different perspectives. Each agent has different blind spots; rotating broadens coverage. The focus areas section and commit history provide continuity so a fresh agent can pick up context without duplicating prior work.
