# Writing for Humans

Open everything a human will read with one short paragraph in plain English that gives the whole picture: what this is, where it stands, and what you want from the reader. Write it so the owner can read it once, cold, without effort, and know enough to decide how much of the rest to read — often nothing. Technical depth is welcome later in a long document; it is never welcome first.

## The opening paragraph is a routing decision

Its job is not to summarize the work but to let the reader allocate attention: act now, skim, read fully, or stop. A summary answers the writer's question — what did I do — while the routing paragraph answers the reader's: what is this, does it work, what do you need from me. Write it for the person who stops after it; they must leave with a correct picture and know what they are choosing to skip.

## General first, specific later

Order every document by depth: the plain outcome, then the shape of the work, then mechanism, then the receipts — paths, commands, logs, exact errors — deepest. Each layer is a valid stopping point that leaves the reader correct at lower resolution. The longer the document, the more strictly this matters; a three-sentence chat reply is one layer, and it is the plain one.

## Length is a cost the reader pays

Optimize for the reader's attention, not the writer's completeness. Every sentence must buy more understanding than it costs, and the size of the work does not entitle the report to match it — the agent that has just done a lot is exactly the agent most tempted to prove it. Cut the narration of effort and the tour of everything examined; keep the result, the risk, and the ask.

## Ordinary words before terms of art

Name a thing in ordinary words at first use; switch to the precise term afterward only when the document needs it repeatedly. Terms that exist only inside this project — worktree flows, R-levels, relay pushes, rail and gate names — are the worst offenders: the writer has just spent hours inside them and mistakes them for shared vocabulary, and the reader has not. If a term would appear only once, keep the ordinary words and drop the term.

## Bad news goes first

A failure, a blocker, an unmet assumption, or a refusal leads the opening paragraph in plain words: this does not work yet, I could not verify X, I did not do Y because Z. Uncertainty too — say what is known, unknown, and guessed before any detail. Burying the point under technical material costs the most attention exactly when the reader must act, and reads as hiding it.

## Plain does not mean imprecise

Lead plainly; never blur a technical fact into a wrong one. Where exactness is load-bearing — a command, an interface contract, a security boundary, a version, an error message — give the exact form verbatim next to its plain statement, and let precision win. A friendly paraphrase that changes the meaning is a defect, not a simplification.

## Where this applies

Everything a human reads: chat replies, reports, docs, review comments, PR and issue bodies, plan sections, commit messages. Artifact shapes stay with their owners — `issue-writing.md` for issue bodies (its plain opening sentence is this rule applied), `dev-plans.md` for plan sections, `git-workflow.md` for PR bodies and commit messages, `memory.md` hygiene for memory files. This file adds no headings, sections, or steps anywhere; it governs the order and the words of what is already being written.
