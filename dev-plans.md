# Dev Plan Guidelines

A dev plan is written only for R4 work (`risk.md`), or when an arc needs irreversible cross-repo cutover choreography that the tracking issue cannot hold. Everything else — R0 through R3, single-repo or multi-repo — is specified by the tracking issue's body (`issue-writing.md`) and reviewed as code through the PR body (`memory.md`, PR Workflow). Writing a plan for R3 work restates the issue and buys a review of prose; the review budget is spent on the diff instead.

When a plan exists it includes these sections. Template: `allod/memory/templates/dev-plan.md`.

1. **Tracking Issue** - Forge issue URL or number for the work. If multiple PRs are expected, state which PR should carry the closing keyword.

2. **Goal** - One sentence describing the user-visible outcome.

3. **Scope** - Which files, services, or modules are in scope and which are explicitly out of scope.

4. **Risk Assessment** - The residual risk level after the planned validation passes, why that level is appropriate, and what human scrutiny is most useful.

5. **Interface Contracts** - Any function signatures or API shapes the agent must match. If none apply, state that explicitly.

6. **Agent Gates** - Actions the agent cannot perform, such as creating repos, deploying, granting permissions, running host-only commands, or changing secrets. State what the human must do and what it blocks. Omit only if none apply.

7. **Acceptance Tests** - Tests the agent can run locally before declaring the task complete. Must be concrete and executable. For NixOS, Home Manager, provisioning, systemd, or wrapper-script changes, include checks against generated lifecycle artifacts and expected absent-resource paths, not only source evaluation or happy-path builds.

8. **Rollback Plan** - How to revert the change if verification fails.

## Plan Shape

A plan holds decisions; knowledge lives elsewhere. Four things earn plan lines:

1. **Unmeasurable decisions** - scope, ordering, ownership, irreversible-step choreography, rollback. No experiment settles these; prose is their native form.
2. **Cross-boundary interfaces** - contracts another repo or slice consumes. A contract nothing external relies on is implementation detail.
3. **Gates** - what the agent cannot do and what the human must.
4. **Acceptance observations** - named witnesses and what each must show, not embedded test scripts.

Evict everything else to where it is cheaper and truer: an empirical claim about upstream becomes a spike measurement recorded on the issue, or a module assertion the plan cites; behavior a boot can answer becomes an observation on a witness (`testing.md`); a failure-mode enumeration becomes the module's assertions, with the plan naming only the class they cover; rationale goes in the issue body.

Soft cap: about 150 lines. Exceeding it requires naming the unmeasurable decision that forces the length, the same way a risk score is justified. Security boundaries and irreversible cutover choreography legitimately spend lines; the cap exists to make the plan spend them there. The first review question on a long plan: which lines are measurements pretending to be decisions?

Complex work slices by witness; no plan grows to hold it. Each slice has its own landable PR set, its own witness that proves it worked, and a stated interface to the next slice. The parent plan is an arc map - ordered slices, about one line each: what it delivers, what witnesses it, what it unblocks. Contracts bind late: they are written in the slice that implements them, when that slice starts, informed by measurements from the slices before it. A parent that pre-binds every downstream contract is specifying what it should be scheduling.

## Risk Assessment

Score residual risk per `risk.md`: the level after planned validation passes, why that level is appropriate, and what human scrutiny is most useful. For multi-PR plans, assign risk per PR or milestone. The score can change during implementation if the diff, validation, or rollback story changes.

## Issue and PR Linkage

- If no tracking issue exists, create one before implementation PRs are opened.
- Add the issue URL or number to the dev plan before implementation starts.
- Reference the issue from every implementation PR.
- For multi-repo or multi-PR work, use `Refs owner/repo#N` on earlier PRs and `Closes owner/repo#N` only on the final integration PR.
- If the final PR changes during implementation, update the dev plan and PR bodies.
- Leave archived strategy dev plans alone unless asked.

## Plan Storage

A plan lives in the repository whose work it describes, so push rights follow the code's boundary (see `agent-behavior.md`): agents push private plans directly and leave public ones for relay. Keep private text out of public history — reset drafts rather than accumulating them.

A cross-boundary plan splits: the private plan owns integration and full-build validation and links to the public one; the public plan is a self-contained leaf with no private references, executable by a public-only agent.

## Plan Review

Only an R4 plan is reviewed as a document, with the pass budget in `risk.md`. The review request is a comment on the plan's PR naming the plan file, the lenses below, and the severity scale; there is no separate review-prompt file. Findings and their disposition are recorded as PR comments, and the plan is edited in place.

### Standing Focus Areas

These six lenses apply to every plan review pass as defaults, and the same six are the reviewer's lenses on an R4 code diff.

Verify prereqs, and delete each one once met: replace an interface prereq with the contract it now provides, and drop a satisfied sequencing gate (e.g. "lands after branch X merges") outright. Do not keep met preconditions as historical notes — a plan is not a log of legacy implementations.

1. **Internal consistency** - Do Interface Contracts, PR descriptions, and Acceptance Tests agree?
2. **Operational sequencing** - Can someone execute the plan cold without getting stuck?
3. **Risk calibration** - Does the residual risk level match the blast radius, rollback path, and validation evidence?
4. **Acceptance test coverage** - Do tests exercise every contract rule? Do all tests trace to a requirement?
5. **Rollback fidelity** - Does the rollback plan undo what the implementation does, including partial states?
6. **Generated lifecycle behavior** - Do generated activation scripts, systemd units, wrappers, provisioning phases, rebuild paths, and missing optional resources behave correctly?

### Agent Rotation

When an R4 review runs more than one pass, rotate models between passes: each has different blind spots. The PR's comments and commit history provide continuity so a fresh reviewer can pick up context without duplicating prior work. Record in the PR how each pass's fixes held up; drop a model from the rotation after repeated same-feature regressions.

### Review Model Pool

The pool serves code reviews as much as plan reviews; the roles below read "plan" as "change". A cross-vendor swap is the strongest rotation available — different training, different blind spots — so prefer it over a same-family model change when the previous pass found little.

Re-check which models the current runner can actually instantiate before each pass: a picker row appears or disappears with a CLI bump (`agent-cli-updates.md`), and an entitlement can drop one without warning. The roster below is the current fleet, not a permanent list.

Invocation per runner. Codex: `codex --model <id>`, effort via `-c model_reasoning_effort='"<level>"'`; headless is `codex exec`. Its `workspace-write` sandbox blocks the network and the nix daemon socket, and a linked worktree's `.git` metadata is read-only there — a codex agent can edit but not build, commit, or push, so the driver validates, commits, and pushes outside. OpenAI-side content filtering can kill a codex session mid-task (observed on credential-leak witness code); edits survive uncommitted in the tree, so inspect and finish rather than restart. Claude Code: `claude --model <alias-or-id>`, effort via `/effort`, `--effort`, or `effortLevel` in `settings.json`. pi: `pi --provider <provider> --model <id> --thinking <level> --no-session -p '<prompt>'` runs headless and can drive the same model ids through a configured model-router provider; wrap long runs in `timeout` — pi never retries a stalled stream, and a stall hangs silently with zero output and zero open connections.

| Model | Runner | Effort ceiling | Role |
| --- | --- | --- | --- |
| `gpt-5.6-sol` | `codex` | `ultra` | Default for R3/R4 plans, cross-repo contracts, security boundaries, generated lifecycle behavior, and terminal verification. |
| `claude-fable-5` / `fable` | `claude` | `max` | Highest-capability Claude option. R4 plans, cross-repo generated lifecycle behavior, and terminal verification. |
| `claude-opus-5` / `opus` | `claude` | `max` | Default for R2/R3 plans and the usual Claude-side rotation partner. |
| `gpt-5.6-terra` | `codex` | `ultra` | Cost-balanced choice for R0-R2 plans and an independent rotation partner for broader first passes. Do not select it when the current review prompt records repeated regressions in the feature under review. |
| `claude-opus-4-8` | `claude` | `max` | Previous Opus. Scoped stability pass on a fix authored by Opus 5, without leaving the tool. |
| `claude-sonnet-5` / `sonnet` | `claude` | `max` | Cost-balanced R0-R2 passes and broad first passes. |
| `gpt-5.5` | `codex` | `xhigh` | Previous frontier model. Rotation partner once the 5.6 family has already reviewed the feature. |
| `gpt-5.4` | `codex` | `xhigh` | R0-R2 passes only. |
| `gpt-5.6-luna` | `codex` | `max` | Mechanical checklist preflight and low-risk, high-volume triage when available. Not the sole terminal reviewer for an R3/R4 plan. |
| `gpt-5.4-mini` | `codex` | `xhigh` | Cheapest. Checklist and metadata passes only, never a terminal reviewer. |
| `claude-haiku-4-5` / `haiku` | `claude` | none | Takes no effort setting. Checklist and metadata preflight only, never a terminal reviewer. |

Rows run strongest role first, so the row order is the selection order once the R level is known. A new runner costs rows plus one invocation clause, never its own table — keying the roster on the tool would leave nothing to hold a model that two runners can both drive. `none` is a real effort ceiling for a model that exposes no reasoning-effort control, not a gap to fill in.

The Claude Code picker collapses superseded rows; a pinned version ID such as `claude-opus-4-8` still resolves, but confirm the runner accepts it before recording it as the next pass's model.

After one model authors a structural fix, use a different eligible model for the scoped stability pass. If the plan excludes every available model, report that no eligible verifier is callable instead of naming or silently substituting an unavailable model.

### Review Effort

Choose the model and reasoning effort separately:

- `medium` is the minimum for R0/R1 checklist, consistency, and metadata review.
- `high` is the default for R2/R3 plans and scoped verification of a structural fix.
- `xhigh` is appropriate for R4 plans, security or privacy boundaries, cross-repo generated lifecycle behavior, or a feature where an earlier `high` pass missed a blocker.
- `max` or `ultra` is exceptional: use it only when the runner exposes that level and the hardest quality-first review is likely to justify the added latency and cost. Compare it with `xhigh` on representative work instead of assuming more effort is better.

The ladder is shared, but its top is model-specific — read the ceiling off the roster above before naming a level. `ultra` exists only on `gpt-5.6-sol` and `gpt-5.6-terra`. Claude Code adds `ultracode` beyond `max`: it pairs top effort with multi-agent workflow orchestration, is offered only on models that support `xhigh`, and is the same class of exception as `max` — justify it, do not default to it. A level the model does not support can fall back silently rather than erroring, so confirm the level that actually applied before recording it.

Record the exact effort in every pass. Hold effort constant when comparing model stability, and do not count a second pass by the same model at a different effort as independent model rotation.

### Review Evidence

Review evidence lives in the PR body's Validation section, not in a separate file and not in a global model leaderboard: the exact model and reasoning effort, the reviewed commit, findings by severity, and what was fixed or discarded and why. Finding more defects is not a negative result; evaluate detection yield separately from the stability of fixes that model authored.

Do not accumulate cross-plan win rates, cost guesses, or raw finding totals in shared memory. Those comparisons are meaningful only under a controlled eval with the same plan snapshots, prompts, tool access, budgets, and scoring rubric; keep that dataset and its versioned results as a separate artifact if it becomes useful.
