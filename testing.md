# Proof and Check Policy

A check pins a property to a witness. Before writing or keeping a check, name the cheapest standing witness that pins the same property:

1. **Live witness** - a real machine, canary or deployed, whose ordinary operation demonstrates the property: it boots, keeps its state, serves its endpoints. Guard it with a smoke script plus the modules' own assertions; do not re-simulate it in fixtures.
2. **Assertion witness** - a fail-closed evaluation assertion inside the module. A check that builds fixture systems only to trigger an assertion the module already enforces is duplication: keep the assertion, delete the fixture check.
3. **Pin witness** - a check that verifies upstream behavior at a pinned revision rather than our own composition. It runs when the pin advances - on the lock-bump path - not on every change.

**Deletion rule:** delete a check when a cheaper witness pins the same property.

**Creation rule:** when a claim can be settled by booting or measuring on a reachable witness, schedule the experiment instead of specifying the outcome and proving the specification by inspection. Specification written ahead of execution is provisional; measurement wins and rewrites it.

Never delete: public-boundary leak scans, module eval assertions, and checks whose property a healthy running system cannot witness because its failure is silent - cross-VM isolation is the standing example.

Sabotage fixtures prove a validator can fail: one per validator, not one per failure mode it rejects.
