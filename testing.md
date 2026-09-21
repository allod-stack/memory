# Proof and Check Policy

A check pins a property to a witness. Before writing or keeping a check, name the cheapest standing witness that pins the same property:

1. **Live witness** - a real machine, canary or deployed, whose ordinary operation demonstrates the property: it boots, keeps its state, serves its endpoints. Guard it with a smoke script plus the modules' own assertions; do not re-simulate it in fixtures.
2. **Assertion witness** - a fail-closed evaluation assertion inside the module. A check that builds fixture systems only to trigger an assertion the module already enforces is duplication: keep the assertion, delete the fixture check.
3. **Pin witness** - a check that verifies upstream behavior at a pinned revision rather than our own composition. It runs when the pin advances - on the lock-bump path - not on every change.

**Deletion rule:** delete a check when a cheaper witness pins the same property.

**Creation rule:** when a claim can be settled by booting or measuring on a reachable witness, schedule the experiment instead of specifying the outcome and proving the specification by inspection. Specification written ahead of execution is provisional; measurement wins and rewrites it.

**A check that only reads a generated artifact witnesses its text, not its behavior.** Grepping a generated unit, script, or config proves the string was written; it cannot prove the command runs. Resolve what the artifact names and execute it in the check. The standing example: a module interpolated a `bin/` path inside a flake input holding source rather than the built package, so the path did not exist. Every assertion over the generated unit passed, and the service failed on every timer run on the machine, where nothing was watching.

Never delete: public-boundary leak scans, module eval assertions, and checks whose property a healthy running system cannot witness because its failure is silent - cross-VM isolation is the standing example.

Sabotage fixtures prove a validator can fail: one per validator, not one per failure mode it rejects.

## Test Code Placement

Production modules contain no test-only branches, fixtures, or hooks. Checks live in their own tree (`checks/`) and exercise the real production generators by injecting fixture *inputs* through the generator's existing parameters. Two failure modes bound the rule from opposite sides:

- **Drift:** a check that reimplements the generated artifact in parallel witnesses its own reimplementation, not production. Consume the same generator; swap only its inputs.
- **Contamination:** a production path that knows about tests (a fixture flag, an `isTest` branch, an embedded test key) puts unexercised or test-serving code on the critical path — worst on paths that handle secrets as root.

When a generator cannot be exercised with fixture inputs, widen its parameters at the composition seam; do not fork it and do not teach it about tests. `modules/pi-provider-lifecycle.nix` in archetypes (production generator, parameterized over the provider catalogue, the credential projection, and the credential runtime root) consumed by `checks/pi-provider-lifecycle.nix` (fixture providers and a runtime root inside the build sandbox, nothing else changed) is the standing example.

**Do not name a real machine in a fixture.** A check that picks a production machine to supply a property, such as a runtime or a machine type, breaks when the fleet changes for unrelated reasons. Written as a negative it stops working altogether once the last machine with that property is removed. Build a fixture machine that carries the property instead.


**A framework check reads the composed machines or its own fixtures — never a data input's contents by name.** A machine name, a credential name, a ciphertext filename, or a check the input is supposed to export are all data a deployment fork replaces, so naming one passes on the template and fails on every fork, whose only recourse is to exclude the check and lose its coverage. Read `machineConfigurations` iterated dynamically, or build the fixture entirely inside the framework repo. The distinction that keeps this usable: a data repo's *file layout* is contract, because production itself reads `${secrets}/git/protected-branches`, while its *names* are not.

Widening a builder's parameters to make that possible opens a second hole. In archetypes `machineConfigurations` merges `profileData` and inventory's `profile` settings into builder arguments, so every new parameter is also a key the profiles layer can now set: `platform`, `secrets` and `hardware` are pinned back after that merge, and `runtime` is deliberately left overridable. A parameter added for a fixture is a production seam too.

The standing witness is `archetypes/checks/fork-fixture/`, a second synthetic data trio sharing no name with the template's, run by `./fork-safety.sh` with `--override-input` on all three, one `nix` process per check. It is a pin witness for the lock-bump path, not a per-commit gate: evaluating the suite twice on every commit doubles a run that already peaks near the memory ceiling. Keep the fixture a plausible fork — tuning fixture data until a check passes is how the witness stops witnessing.

**Two checks with the same derivation name shadow each other.** Merged check sets keep the last one, so a fork's local check silently replaces the framework's if both use the same `runCommand` name. Nothing warns, and the replaced check never runs again. Assert names are unique after merging.
