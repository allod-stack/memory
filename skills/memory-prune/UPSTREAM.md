# Upstream

- Project: backpass
- Repository: https://github.com/kunchenguid/backpass
- Revision: `f16b75bb23ff6d6dac9f72a7bb6dc5e79583c477`
- License: MIT, reproduced in `../memory-backpass/UPSTREAM.md`

`score-prompt.md` is a trimmed derivative of the sibling `memory-backpass` skill's `analysis-prompt.md`, which adapts `src/prompts/analysis.md` from backpass. The gap-finding half of that prompt is dropped: this pass asks only which instructions changed what the agent did. The procedure in `SKILL.md` has no upstream counterpart.
