---
name: explain-diff
summary: Render a code change as a rich, self-contained HTML explanation
description: Create a rich, interactive, self-contained HTML explanation of a code change. Diff, commit range, branch, or pull request. Use when a user asks to understand or explain a change in depth.
---

# Explain Diff

Create a rich, interactive explanation of the specified code change.

## Investigate first

1. Resolve exactly which diff, commit range, branch, or pull request the user means. If it is ambiguous and cannot be inferred safely, ask.
2. Read the complete diff.
3. Explore enough surrounding code, documentation, history, and tests to explain both the existing system and why the change was made. Do not merely paraphrase changed lines.
4. Distinguish facts visible in the repository from interpretations. Do not invent intent.

## Content

Build one coherent page with these sections:

- **Background:** Explain the relevant existing system. Start with beginner-friendly context that experienced readers can skip, then narrow to the concepts directly needed to understand the change.
- **Intuition:** Explain the essence of the change before its mechanics. Use concrete examples and small toy data. Use figures and diagrams where they clarify the idea.
- **Code:** Walk through the changes at a high level. Group them by concept or execution flow rather than blindly following file order. Connect implementation details to behavior and tests.
- **Quiz:** Include five challenging multiple-choice questions that test whether the reader can apply and reason about the change, not merely recall prose. Clicking an answer must reveal whether it is correct and explain why.

### Write a real comprehension test

Draft the quiz only after the explanation is complete. For each item:

1. Choose one important learning objective from the change: predict behavior, trace data or control flow, select the right design under a new scenario, diagnose a plausible failure, or distinguish the implemented guarantee from a tempting overclaim.
2. Write the answer and rationale before writing distractors. The stem must be answerable without seeing the options and must have one defensibly best answer.
3. Create plausible distractors from real misconceptions: the old behavior, a nearby but incorrect component, a reversed condition, a missing edge case, or a solution that works in the happy path but violates one constraint. Misdirection may test understanding, but never rely on obscure wording, trivia, or gotchas.
4. Keep options parallel in grammar, specificity, technical register, and approximate length. Do not make the correct answer consistently longer, more qualified, more precise, or more polished. Avoid answer-signaling words and patterns such as `always`/`never`, repeated phrases from the stem, or one option that alone fits grammatically.
5. Avoid `all of the above`, `none of the above`, double negatives, trick wording, and irrelevant difficulty. Prefer positive stems; if a negative is essential, emphasize it clearly.
6. Make feedback instructional: explain why the selected answer is right or wrong and identify the underlying concept, rather than merely revealing the key.

Use answer positions A–D exactly once each across the first four questions. For the fifth, select A–D with a runtime random source (for example, `shuf` or a language RNG), record the selected position while authoring, and place the correct option there. Then shuffle or deliberately assign distractors around those fixed keys. Do not use intuition to "mix them up." In the HTML, store correctness on the option itself rather than assuming a fixed button index.

After drafting, run an answer-choice audit without looking at the keys: check whether option length, detail, grammar, tone, or repeated vocabulary gives the answer away. Revise any item where a test-wise reader could guess correctly without understanding the change. The set should include at least three application, prediction, diagnosis, or comparison questions; at most two may be direct recall.

Add a concise summary of the change near the top when that helps orient the reader. Mention important tradeoffs, edge cases, or residual risks when the evidence supports them.

## Output

- Write a single self-contained HTML file with inline CSS and JavaScript. Do not use external assets or dependencies.
- Make it one long page with section headings and a table of contents; do not use tabs for the top-level structure.
- Use responsive styling that remains readable on a phone.
- Save it where the deployment keeps standalone reports (the memory index names the repository and path if there is one), as `<topic-slug>-explanation.html`, following that repository's naming conventions. Do not put it in the repository whose diff is being explained.
- Treat the report as a durable standalone artifact: write it so it can be understood months later without session context.
- Check the report for secrets, credentials, private infrastructure details that do not belong there, and accidentally embedded environment data before committing it.
- Commit and push the report directly to that repository's default branch so the user can render it on the forge. Do not leave the only copy in `/tmp` or another VM-local path.
- Tell the user the repository-relative output path, commit, and Forge URL when available.
- Write clear, engaging classic prose with smooth transitions. Do not imitate or claim authorship by a living writer.

## Visuals and code

Choose a small number of consistent diagram families and reuse them. Good options include:

- a simplified representation of the user interface for UI changes;
- a system diagram showing components, communication, and representative example data;
- before/after flows for behavioral changes.

Do not use ASCII diagrams. Build diagrams from semantic HTML and CSS. Use HTML lists for lists.

Use `<pre><code>` for code blocks. Ensure every code block preserves formatting through `white-space: pre` or `white-space: pre-wrap`. Escape repository content before embedding it in HTML.

Use callouts for key concepts, definitions, important edge cases, and cautions. Keep decorative styling subordinate to the explanation.

## Validate before finishing

1. Confirm the file is in the deployment's reports location and follows its naming convention.
2. Confirm the HTML is self-contained and has no external resource references or network requests.
3. Confirm the table-of-contents links and quiz interactions work.
4. Scan every code block and verify its effective CSS preserves whitespace with `pre` or `pre-wrap`.
5. Confirm the explanation covers the actual diff and does not state unsupported intent as fact.
6. Scan for secrets and irrelevant environment-specific data before committing.
7. If a browser is available, open the file or otherwise perform a lightweight render/syntax check.
8. Audit the quiz mechanically: print the five correct-answer letters and option word counts; verify A–D each appears in the first four, the fifth position came from a runtime random source, no position pattern is accidental, and correct options are not systematically longest.
9. Review every distractor against a specific plausible misconception and confirm at least three questions require application, prediction, diagnosis, or comparison.
10. Commit and push the report, then verify the pushed commit.

Adapted from Geoffrey Litt's `explain-diff-html` skill gist: https://gist.github.com/geoffreylitt/a29df1b5f9865506e8952488eac3d524

Quiz-writing guidance synthesized from:

- Vanderbilt University Center for Teaching, "Writing Good Multiple Choice Test Questions": https://cft.vanderbilt.edu/guides-sub-pages/writing-good-multiple-choice-test-questions/
- Yale Poorvu Center for Teaching and Learning, "Multiple Choice Questions": https://poorvucenter.yale.edu/MultipleChoiceQuestions
