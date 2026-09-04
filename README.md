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
<topic>.md             topic files — indexed with one-line descriptions in memory.md
adapters/              tool-specific entry points that redirect to memory.md
  claude/CLAUDE.md
  codex/AGENTS.md
  pi/AGENTS.md
templates/             blank scaffolds referenced by dev-plans.md
  dev-plan.md
  plan-review-prompt.md
skills/                portable skills, one directory each, indexed in memory.md
  fleet-diff/SKILL.md
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

Available skills:

- `fleet-diff` — gates a merge against which machines it actually rebuilds, via `fleet-diff`, the per-machine flake drvPath comparison tool in `tools`.

## Adapters

Each coding agent tool has its own native memory filename. A tool's native
memory config points at `adapters/<tool>/<file>` here — a small entry point in
that tool's format (`CLAUDE.md` for Claude, `AGENTS.md` for Codex and Pi). Each
entry point carries the conversation-start instruction itself and names
`memory.md` directly, so reaching the index costs one file read. That sentence is
repeated in all three deliberately: routing them through a shared include trades
one repeated sentence for an extra file read at the start of every session, and
adds a relative hop an agent can misresolve into skipping memory entirely.
Tool-specific policy that cannot live in memory stays in the adapter (e.g. the
Claude adapter's attribution ban).

## Memory hygiene

Rules that keep this repo a statement of current state rather than a log
(stated in `memory.md`):

- `memory.md` is the only root memory file; everything else is a topic file, adapter, or template.
- Add durable memory to the topic file that owns it; update the index only when adding a new topic file.
- Record state, not a changelog. Memory is the current state of the world plus the decisions and gotchas that constrain future work — git and the forge already log every merge and close.
- Retire landed work. When the work an entry tracks goes terminal, the edit compresses it to its one durable fact or deletes it.

## Memory vs strategy

This repo owns durable, cross-session conventions, workflows, and gotchas — the
material agents need every session. It does not own active dev plans, review
prompts, brainstorms, or user stories; those live in `strategy`. Only the blank
plan and review-prompt scaffolds under `templates/` live here.

## Related repos

- `strategy` — active dev plans, review prompts, brainstorms, and archives; consumes the `templates/` scaffolds here.
- `tools` — the `allod`, `forge`, and workspace CLIs the workflows here invoke.
- `vm`, `archetypes`, `profiles`, `nexus`, `inventory`, `secrets` — the framework and consumer repos whose conventions the topic files document.

## Cloning

    git clone https://forge.anarch.diy/allod/memory.git
