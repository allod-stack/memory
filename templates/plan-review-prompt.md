# [Plan Name] Review Prompt

You are the absolute best at what you do. Write a punchy, funny intro that hypes the reviewer as a [domain expert in relevant technical domains]. Make them feel like a boss. Then tell them not to hold back.

## Your Task

Review the [plan name](plan-file.md) for gaps, misunderstandings, bugs, and flaws. Be direct and specific. Flag anything that will block implementation, create unnecessary work, or leave a landmine for future changes.

Read the actual codebase to ground the review in reality. Do not review the plan in isolation.

## Project Context

**[Project name]** is [one-sentence project description].

Key repos in play:
- `repo-a` - [what it owns]
- `repo-b` - [what it owns]

Current state (verify against the tree before trusting it; line numbers and file layout drift):
- [Relevant architectural facts the reviewer needs to orient]
- [Recent changes that affect the plan]
- [Execution constraints, e.g., "agents run in dev VMs only"]

## Structural Conformance

Before diving into focus areas, verify the plan includes all required sections from `dev-plans.md`: Tracking Issue, Goal, Scope, Risk Assessment, Interface Contracts, Agent Gates, Acceptance Tests, and Rollback Plan. Agent Gates may be omitted only if no actions require a human. If the work spans multiple PRs, verify it assigns residual risk per PR or milestone. If the plan closes its tracking issue, verify which PR carries the closing keyword; a prerequisite plan whose PRs intentionally carry no closing keyword is conformant when it says so.

## Focus Areas

Concentrate your review on these areas where the plan is most likely to have problems. These are lenses, not checklists - follow the thread wherever it leads.

The six standing lenses in `dev-plans.md` (internal consistency, operational sequencing, risk calibration, acceptance-test coverage, rollback fidelity, generated lifecycle behavior) apply as defaults on top of the plan-specific areas below.

1. **[Short name].** [Describe the concern. Explain why it is risky. Pose a direct question about what could go wrong or what is missing.]

2. **[Short name].** [Same structure - name it, explain the risk, ask the question.]

Do not re-open focus areas addressed in previous passes unless the current plan contradicts itself.

## Review Guidelines

- **Forward momentum is king.** Do not nitpick style or suggest nice-to-haves. Only flag things that will actually cause problems.
- **No backwards compatibility required.** This is pre-alpha. We can break any interface.
- **Do not overengineer.** If the plan introduces abstraction that is not needed yet, call it out. Three similar lines beat a premature helper function. Run an explicit SIMPLIFY sweep every pass: actively hunt for scope, ceremony, or abstraction to delete rather than treating a quiet pass as nothing to cut.
- **Solo project, one human.** No team coordination overhead. No release process. No migration guides for other consumers.
- **Security matters, ceremony does not.** The privacy and security boundaries must be airtight. Everything else can be pragmatic.
- **Solve problems as they come.** If the plan front-loads work for hypothetical future needs, flag it.
- **Think operationally.** Consider what happens when someone executes this plan with incomplete context or in the wrong order.
- **Calibrate residual risk.** The risk score is for human triage after validation passes. Challenge understated or overstated risk using blast radius, rollback fidelity, and validation evidence.
- **Inspect generated lifecycle artifacts.** For NixOS, Home Manager, provisioning, systemd, or wrapper-script changes, do not stop at source evaluation. Inspect the generated activation scripts, units, wrappers, install/boot phases, and negative paths for missing optional secrets, credentials, files, or host state.

The person implementing this is technically sharp. They do not need hand-holding; they need the sharp edges they missed.

## Severity Rubric

Use `[BLOCKER]` only when following the plan literally is likely to:

- perform a destructive or unsafe operation;
- fail before implementation can complete;
- leave the resulting system nonfunctional;
- break first boot, activation, provisioning, rebuild, rotation, or rollback lifecycle behavior;
- violate a security or privacy boundary; or
- require missing human input that cannot be inferred from the repo or memory.

Use `[GAP]` for missing or contradictory plan details that could cause rework, test blind spots, stale docs, or implementation ambiguity, but where a competent agent with workspace memory could still proceed safely.

Use `[GAP]` when the Risk Assessment is missing, materially understated, materially overstated, or unsupported by the acceptance tests and rollback plan. High residual risk is not a blocker by itself; it should drive better validation, clearer rollback, or a human gate only when a real human-only action or decision exists.

Use `[SIMPLIFY]` for unnecessary scope, ceremony, or abstraction. Commit SIMPLIFY fixes when they remove implementation work, delete unnecessary scope, or prevent an unnecessary abstraction. Do not create plan commits for wording-only simplifications unless the wording changes execution behavior.

Use `[QUESTION]` only when the plan cannot be corrected from repo context. If the answer is inferable, resolve it as `[GAP]` or `[SIMPLIFY]`.

Do not classify duplicated workspace policy, phrasing improvements, or reminders already covered by memory as findings unless the plan directly contradicts that policy.

## Deliverable

The deliverable is not a report. Every review pass ends with:

1. Plan-file commits for findings that require plan changes, or an explicit no-findings result.
2. A final review-prompt commit updating this prompt's Focus Areas and pass metadata.
3. A push to the remote.

For each finding that requires a plan change, edit the plan and commit the fix. Group changes into logical, self-contained commits.

A one-line commit is fine when it records a real implementation decision. Fold or skip commits that only rephrase already-correct guidance.

## Output Format

Give your review as a numbered list of findings, each tagged as one of: `[BLOCKER]`, `[GAP]`, `[SIMPLIFY]`, `[QUESTION]`. Start with blockers, end with questions. Be blunt.

If a design decision is sound, say so briefly — do not damn with faint praise. If something is right, name it and explain why so a later pass does not undo it. QUESTIONs must be resolved in the plan, not left as open items. If the answer is clear from the codebase, update the plan and commit. If the answer requires human input, add the question to the Focus Areas section for the next pass.

## After Each Pass

As a final commit, update this prompt's Focus Areas section:

1. Remove focus areas that were fully addressed.
2. Refine any focus areas that were partially addressed.
3. Add new focus areas discovered during the review.

The focus areas should always reflect the most productive targets for the next review pass, not a historical record of past ones.

Include a plain-text findings summary in this commit's message:

- Count only findings new to this pass, by tag. Carried-over unresolved items are not findings; move them to Focus Areas rather than re-counting them, so the per-pass counts track a real severity trend instead of inflating it.
- Give each new finding a numbered entry with its tag, short title, one-sentence explanation, fixing commit hash, and issue link if one exists.
- Classify each finding's origin: an original-plan defect, or introduced by an earlier review pass (name the commit that introduced it). Origin is what makes the convergence heuristic below and any trend read possible.
- State what the SIMPLIFY sweep considered for deletion this pass, even when nothing was cut. Two or more consecutive passes with zero SIMPLIFY on a growing plan is a smell to call out, not a clean bill: pure accretion is what breeds internal contradictions.
- Put a final `Model: <exact model>` footer at the bottom (e.g., `Model: claude-opus-4-6`, `Model: gpt-5.5`). Use the exact model identifier, not the agent framework or product name. "Codex", "Claude Code", etc. are agent software, not models. This is review-pass metadata, not authorship attribution.

When a pass commits a structural or design change (a blocker-level fix), the next pass should be a scoped diff review of that change, not a full re-review, run by a different model than the one that wrote the fix — structural fixes are where new blockers enter, and the author model tends not to catch its own gaps. Passes are launched manually one at a time, so you cannot pick or start the next model yourself; make the handoff explicit instead. In the Focus Areas update, add a `Next pass:` line that names the commit(s) to review, says whether it is a scoped diff or a full pass, and recommends a model other than the fix's author (preferably the most fix-stable one on record; see Agent Rotation in `dev-plans.md`). Whoever starts the next pass reads that line and the previous `Model:` footer and picks accordingly. Record in the same update how the fix held up so its stability stays traceable.

Stop the plan-text review when either condition holds:

- Review-introduced findings outnumber original-plan findings for two consecutive passes.
- Two consecutive passes produce no BLOCKER and no original-plan GAP.

At that point the plan text has converged; hand any remaining focus areas to implementation review, and resolve remaining SIMPLIFYs during implementation. The old zero-BLOCKER/zero-GAP/zero-QUESTION rule could grind a token budget to nothing without terminating, because each accretive fix tended to seed the next finding.

## Before Final Response

- Plan fixes are committed, or the pass explicitly found no plan changes.
- This review prompt's Focus Areas are updated and committed.
- The final review-prompt commit message includes the findings summary and `Model:` footer.
- The repo is pushed to the remote.
- `git status` is clean.
