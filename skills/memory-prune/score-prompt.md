<!-- memory-backpass:self -->
You are scoring one distilled agent-session trace against two memory indexes. The only question is which instructions visibly changed what the agent did in this session. Do not propose new instructions, do not reword existing ones, and do not review the code the session produced.

## Instruction indexes

Every line has a run-local stable id in brackets. Public-memory ids begin `PUB-`; private-memory ids begin `PRV-`. Refer to an instruction only by its id.

{{INSTRUCTION_INDEX}}

## Distilled trace

Path: {{TRACE_PATH}}

{{TRACE}}

## Return value

Return one JSON object and nothing else. Do not add prose or a Markdown fence.

```
{
  "sightings": [{"id": "PUB-001", "class": "steered|ignored|harm", "moment": "turn 2", "effect": "what the instruction changed, or what happened instead", "quote": "verbatim text from the trace"}]
}
```

Rules, in order of importance:

1. Every sighting needs a `quote` copied verbatim from this trace. Downstream code whitespace-folds it and discards the sighting unless it is a substring of the whitespace-folded trace. A paraphrase is not evidence.
2. `steered` means the trace shows the agent doing the specific thing the instruction asks. A good outcome that the instruction did not visibly cause is not a sighting; neither is an agent restating the instruction without acting on it.
3. `ignored` means the instruction's subject came up in this session and the agent did not follow it. The instruction being irrelevant to this session is not `ignored` — it is no sighting at all.
4. `harm` means the instruction was followed and that cost something. Never label a skipped instruction `harm`.
5. Return at most one sighting per instruction. If an instruction was followed at one moment and ignored at another, report the `ignored` one, which is the more useful signal.
6. An empty array is valid and common. A session touches a small part of the index, and silence about an instruction is the measurement this pass needs — do not pad the array to look thorough, and never guess an id that is not in the index above.
