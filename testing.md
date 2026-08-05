# Proof and Check Policy

A check pins a property to a witness. Before writing or keeping a check, name the cheapest standing witness that pins the same property:

1. **Live witness** - a real machine, canary or deployed, whose ordinary operation demonstrates the property: it boots, keeps its state, serves its endpoints. Guard it with a smoke script plus the modules' own assertions; do not re-simulate it in fixtures.
2. **Assertion witness** - a fail-closed evaluation assertion inside the module. A check that builds fixture systems only to trigger an assertion the module already enforces is duplication: keep the assertion, delete the fixture check.
3. **Pin witness** - a check that verifies upstream behavior at a pinned revision rather than our own composition. It runs when the pin advances - on the lock-bump path - not on every change.

**Deletion rule:** delete a check when a cheaper witness pins the same property.

**Creation rule:** when a claim can be settled by booting or measuring on a reachable witness, schedule the experiment instead of specifying the outcome and proving the specification by inspection. Specification written ahead of execution is provisional; measurement wins and rewrites it.

Never delete: public-boundary leak scans, module eval assertions, and checks whose property a healthy running system cannot witness because its failure is silent - cross-VM isolation is the standing example.

Sabotage fixtures prove a validator can fail: one per validator, not one per failure mode it rejects.

## Test Code Placement

Production modules contain no test-only branches, fixtures, or hooks. Checks live in their own tree (`checks/`) and exercise the real production generators by injecting fixture *inputs* through the generator's existing parameters. Two failure modes bound the rule from opposite sides:

- **Drift:** a check that reimplements the generated artifact in parallel witnesses its own reimplementation, not production. Consume the same generator; swap only its inputs.
- **Contamination:** a production path that knows about tests (a fixture flag, an `isTest` branch, an embedded test key) puts unexercised or test-serving code on the critical path — worst on paths that handle secrets as root.

When a generator cannot be exercised with fixture inputs, widen its parameters at the composition seam; do not fork it and do not teach it about tests. `modules/microvm-credential-hook.nix` (production generator, parameterized over ciphertext root, identity paths, and registry material) consumed by `checks/microvm-credential-hook.nix` (fixture inputs only) is the standing example, mirroring nexus `launcher.nix`'s sabotage-variant pattern.
