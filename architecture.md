# Allod Architecture Principles

> An allod is land held in absolute ownership — no rent, no fealty, no master.

Allod is a self-sovereign NixOS stack: one human owner, a bare-metal host that provisions disposable VMs, a self-hosted forge, and AI agents working as first-class citizens inside those VMs. This file is the constitution for design work. Check every architecture or design decision of consequence against these principles before writing a dev plan or accepting a design. A decision that violates a principle needs either a different design or an explicit, recorded owner ruling — never a silent exception.

These principles are distilled from the framework code, the full commit history, the strategy corpus, and the forge issue and PR record. They describe decisions that were made deliberately, usually more than once, and often after paying for the alternative.

## When Principles Conflict

1. The privacy and secret boundaries are airtight. This outranks everything else.
2. Human authority over irreversible and trust-changing acts is structural. It is never relaxed case-by-case.
3. After those two, forward momentum is king: security matters, ceremony does not.

## Sovereignty

**1. Own the root; treat everything else as swappable.**
Every load-bearing dependency — forge, host, provisioning, backups — runs on hardware and services the owner controls. Third parties are integrations, never foundations: "Forge hosts are integrations, not foundations. So are coding agents and model routers." Nothing in the stack may require GitHub, AWS, or any service the owner cannot replace. Availability comes from tested, age-encrypted offsite backups (the offsite host sees only ciphertext), not from managed redundancy. A backup you have never restored is untested.

**2. One human is the root of trust.**
The stack is single-user by design: the owner's identity is the role. Every trust chain terminates at the human at the host terminal — sole holder of the master age/SSH identity, signing keys, merge rights, and provisioning commands. Genericity for other people comes from forking the framework, not from adding accounts or multi-tenancy.

**3. Build public; deploy private.**
The framework exists to be forked and audited: the code is the architecture. Public repos must be runnable by strangers — synthetic but buildable template data, one concrete machine per archetype, no credentials required to clone, evaluate, or build. Strong copyleft keeps it a commons. The adversary is platform dependence, data brokers, and bulk surveillance — not state-grade attackers; do not pay state-grade costs.

## Boundaries

**4. A boundary that depends on cooperation is not a boundary.**
Enforce every rule at the deepest layer that cannot be talked out of it: capability absence and server-side permissions, then cryptography, then git hooks, then tool defaults, then templates, then prose — each outer layer only a backstop for the one beneath. Hooks are guardrails against agent error; key custody, account scoping, and absent capabilities (the forge CLI has no merge command) are walls against agent compromise. Tools ship no bypass flags: overriding a rail is a deliberate human act with standard git, never a mode an agent can learn.

**5. Code, data, and keys have different owners; never mix them.**
Framework repos carry zero identity: public repos describe machines, private keys activate them. Inventory owns machine facts, secrets own identity and trust roots, framework repos own behavior, and they compose only through declared interfaces. Real component names are fine in public; identity material never is; age-encrypted blobs are fine in public repos by design. Material found in the wrong layer migrates toward the more private owner — that ratchet turns one way only.

**6. Privilege flows downhill: human, then host, then VM, then agent.**
The host provisions, rebuilds, rotates, and repairs; VMs never gain host authority; agents never merge, sign, provision, or publish. No single environment holds both private and public capability — only the human bridges compartments, through reviewed artifacts such as patches and relayed pushes. Humans and agents are separate principals with separate accounts and separately scoped tokens.

**7. The VM is the cage; inside it, the agent runs free.**
Containment comes from the VM boundary and credential scoping, not from agent-side permission prompts — those are disabled on purpose. Per-VM, per-purpose credentials bound any compromise to that machine's own access. Capabilities are allocated by purpose, and absence is policy, not oversight: privacy VMs get no forge access, no git, no python. Policy lives in topology, not in application config: Tor-only egress is a fail-closed kill switch that holds regardless of the application's own configuration; no shared folders; SSH only; host keys are pinned from registry data — never TOFU.

## State and Truth

**8. One source of truth per fact; consumers derive.**
Machine facts live in inventory, identity in secrets, behavior in framework code; everything downstream is generated and validated by checks that fail closed. Duplicating a fact is a bug even when copying is convenient. Only tracked, declarative sources exist to the build — gitignored files are invisible at eval time, so there is no side channel for configuration.

**9. Everything rebuilds from a clone; only keys are precious.**
A machine is a pure function of framework plus inventory plus secrets: disk layout, system, home, and workspace all regenerate from the forge. VMs are disposable and destructively replaceable, never repaired by hand. The backup-worthy set is exactly the forge and the encrypted key material — if losing something else would hurt, it belongs in a repo.

**10. Published is forever.**
Public history is never force-pushed, re-rooted, or scrubbed after the fact, and exports get fresh history with no connection to private history. Anything exposed is assumed seen: a leak triggers rotation, not deletion. Sanitize by regeneration — synthetic values built to the same schema and verified by ban-token scans — never by redaction. Rotation is a routine, staged lifecycle (stage, activate, retire), tooled and rehearsed, so revoking anything is cheap the day it matters.

**11. Fail loud; never fall back silently.**
Unknown types, duplicate keys, and missing definitions are evaluation errors, not silent defaults; replacing public behavior requires an explicit override marker. Destructive tooling preflights every precondition and touches nothing on failure; when state is left uncertain, it says so loudly and preserves the evidence. Validation is itself validated: a check that cannot be shown to fail on sabotaged input does not count.

## Process

**12. Every significant change justifies itself before it lands.**
A tracking issue exists first. The plan states scope, interface contracts, agent gates, acceptance tests, and rollback, and carries a residual risk score (R0–R4) keyed to authority of state, blast radius, and recoverability. Risk raises scrutiny — sharper tests, tighter rollback, another review pass — it does not block by itself.

**13. Believe generated artifacts, not source review.**
Acceptance tests are concrete and executable, and must exercise generated lifecycle behavior — activation scripts, systemd units, provisioning phases, rebuild paths — including negative paths and absent optional resources. A happy-path build can pass while first boot strands the machine; that lesson was paid for once and is not to be repurchased.

**14. Lessons become mechanism; agent context is a wasting asset.**
A rule that must survive a long session moves out of prose and into tools and hooks — recurring failures get promoted up the enforcement ladder, because memory guidance degrades as context fills. Judgment calls deliberately stay prose. Gates that cannot verify quality get deleted rather than elaborated: security matters, ceremony does not.

**15. Move in small, independently landable steps.**
Decompose monoliths so each step has its own blast radius and rollback. Extract on second use — three similar lines beat a premature helper. Tooling priorities, in order: minimalism, security, usability, maintainability, simplicity; dependencies are liabilities before they are conveniences. Review plans adversarially with rotated models until convergence, then stop — pure accretion breeds contradictions. Archived plans are historical record; never retro-edit them.

---

Distilled 2026-07 from all public and private allod repositories. When a settled decision contradicts this document, update the document through the normal change process and state the contradiction — this file loses to reality, but only out loud.
