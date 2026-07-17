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
<topic>.md             eleven topic files (see Topic files)
adapters/              tool-specific entry points that redirect to memory.md
  claude/CLAUDE.md
  codex/AGENTS.md
  pi/AGENTS.md
templates/             blank scaffolds referenced by dev-plans.md
  dev-plan.md
  plan-review-prompt.md
```

## Topic files

| File | Contents |
|---|---|
| `allod.md` | Allod overview, Forgejo org, repo inventory, `allod/strategy` subdirs, and `forge` CLI / workspace session notes. |
| `architecture.md` | The core architecture principles — the constitution to check every design decision of consequence against. |
| `git-workflow.md` | Branching, the `allod change` begin/record/submit flow, `forge` CLI, and issue/PR linkage and body rules. |
| `issue-writing.md` | Implementation-issue scope and structure; user-story-first bodies; cross-repo story tracking. |
| `dev-plans.md` | Required dev-plan sections, the R0–R4 residual-risk rubric, and the iterative plan-review process. |
| `security-practices.md` | Token handling — keep tokens out of argv, prefer stdin, build narrow credential helpers. |
| `agent-behavior.md` | Scope discipline and handling user interruptions. |
| `vm-tooling.md` | VM package policy (`jq` on all dev VMs; no `python3` on privacy VMs). |
| `vm-provisioning.md` | Provisioning stack repos, source-of-truth pointers, and provisioning gotchas. |
| `nix.md` | NixOS gotchas — read-only `nix.conf`, netrc/libgit2, disko, agenix, SSH-key handling. |
| `age.md` | age / agenix workflows — safe secret input, running agenix, recipient keys. |

## Adapters

Each coding agent tool has its own native memory filename. A tool's native
memory config points at `adapters/<tool>/<file>` here — a small entry point in
that tool's format (`CLAUDE.md` for Claude, `AGENTS.md` for Codex and Pi) whose
only job is to redirect to `../../memory.md`. So each tool needs one small
native-format file and the durable content lives once. Tool-specific policy
that cannot live in the shared files stays in the adapter (e.g. the Claude
adapter's "never add AI attribution").

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
