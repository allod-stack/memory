# allod/memory

Version-controlled workflow memory for Allod coding agents. This repo is the
durable, git-tracked memory that agents read at the start of every session to
learn Allod's conventions, workflows, and gotchas. Agents read the root index
`memory.md` first, then the topic files it points to that are relevant to the
task.

Allod is a self-sovereign NixOS VM stack for agentic coding and privacy tasks.
Memory holds the durable conventions and workflows; short-lived planning
material (dev plans, review prompts, brainstorms) lives in `strategy`.

## Layout

```
memory.md              root index — agents read this first, then the topic files it lists
ledger.md              sightings waiting for corroboration; not read at session start
<topic>.md             topic files — indexed with one-line descriptions in memory.md
.hooks/                tracked repository hooks, including the memory byte-budget gate
templates/             the dev-plan scaffold referenced by dev-plans.md (R4 plans only)
  dev-plan.md
skills/                portable skills, one directory each, indexed in README.md
  delegate-implementation/SKILL.md
  fleet-diff/SKILL.md
  forge-groom/SKILL.md
  list-models/SKILL.md
  list-models/probe-models
  memory-backpass/SKILL.md
  pick-next-issue/SKILL.md
```

## Skills

Memory tells an agent what the conventions are; a skill tells it how to drive
one specific tool well, at the moment it reaches for that tool. Skills follow
the [Agent Skills](https://agentskills.io/) directory format: each lives at
`skills/<name>/SKILL.md` with a frontmatter `name` and `description`. Point a
compatible harness at this repository's `skills/` directory, or link individual
skill directories into the harness's own skill location.

A skill earns a place here when its subject is a tool the workflows in this repo
invoke and getting it wrong is expensive or silent. Anything narrower than that
belongs with the tool, and anything broader is a topic file.

Every description opens with a one-line lead: a first sentence of at most 100
characters that says what the skill is. The lead is all a skill list shows —
`allod skill` truncates there — so it must stand alone as the skill's identity.
Trigger material ("Use when …") and any further detail follow the lead in the
same description; the text is reordered, never dropped.

Available skills:

- `delegate-implementation` — one overseer on the most capable model, waiting on callbacks instead of polling, with cheaper workers in their own worktrees and reviewers from another model family; how to scope a worker, keep the overseer's context small, and what the overseer runs itself before a PR body claims it.
- `fleet-diff` — gates a merge against which machines it actually rebuilds, via `fleet-diff`, the per-machine flake drvPath comparison tool in `tools`.
- `forge-groom` — triages every open issue and PR in the allod repos against master, closes only what a merged `Closes` proves settled, labels the rest, and writes one short report; the weekly timer and the by-hand run share it.
- `list-models` — where each installed agent CLI (pi, codex, claude) keeps its model list, why a listed model is not a served one, and `probe-models`, which lists what is present and verifies only the models you name.
- `memory-backpass` — reads recent distilled session traces against both memory indexes and opens one evidence-backed private-memory PR, weekly or by hand.
- `pick-next-issue` — sweeps every open issue in the repos the agent can push to, verifies the shortlist against master, picks the one worth implementing next, explains it and the choice in plain English, and asks the owner before starting.

## Harness bootstraps

Sessions reach `memory.md` in one hop: each VM's harness bootstrap file
(`~/.claude/CLAUDE.md`, `~/.codex/AGENTS.md`, `~/.pi/agent/AGENTS.md`) is
generated at VM build time by the allod/archetypes home-manager module, from
the inventory registry's memory-marked repositories, and names the absolute
`memory.md` path of every memory checkout the machine clones. This repo keeps
no per-tool entry points; tool-specific policy that cannot live in memory
(such as Claude's attribution ban) lives in the generator's Claude template.

## Memory hygiene

Rules that keep this repo a statement of current state rather than a log
(stated in `memory.md`):

- `memory.md` is the only root memory file; everything else is a topic file or template.
- Add durable memory to the topic file that owns it; update the index only when adding a new topic file.
- Record state, not a changelog. Memory is the current state of the world plus the decisions and gotchas that constrain future work — git and the forge already log every merge and close.
- Retire landed work. When the work an entry tracks goes terminal, the edit compresses it to its one durable fact or deletes it.
- Keep the always-loaded surface within the tracked hook's byte caps: 16KB for `memory.md`, 24KB per topic file, and no ISO date such as `2026-09-08` in `memory.md`; move detail rather than raising a cap.
- Put one session's lesson in its topic file, or in `ledger.md` when no topic owns it; never put it straight into `memory.md`.
- Add an index line only after two distinct-session sightings in `ledger.md`, unless the reason it is safety-critical is stated.
- Remove an index line only when following it caused harm; reinforce or sharpen an ignored line instead.
- Keep `ledger.md` out of session-start reads; `memory-backpass` promotes and prunes it.

## Memory vs strategy

This repo owns durable, cross-session conventions, workflows, and gotchas — the
material agents need every session. It does not own active dev plans, review
prompts, brainstorms, or user stories; those live in `strategy`. Only the blank
dev-plan scaffold under `templates/` lives here.

## Related repos

- `strategy` — active R4 dev plans, brainstorms, and archives; consumes the `templates/` scaffold here.
- `tools` — the `allod`, `forge`, and workspace CLIs the workflows here invoke.
- `vm`, `archetypes`, `profiles`, `nexus`, `inventory`, `secrets` — the framework and consumer repos whose conventions the topic files document.

## Cloning

    git clone https://forge.anarch.diy/allod/memory.git
