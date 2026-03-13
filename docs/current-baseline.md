# Current Baseline

## Task
Lore-safe proofreading

## Baseline Model
qwen2.5:14b

## Baseline Prompt
prompts/lore-safe-proofread-003.md

## Baseline Input Reference
inputs/lore-safe-test-001.md

## Best Confirmed Run
run-2026-03-13-005

## Latest Successful Verification Runs
- run-2026-03-13-014
- run-2026-03-13-015

## Locked Conclusion
The accepted baseline remains unchanged:

- model: `qwen2.5:14b`
- prompt: `prompts/lore-safe-proofread-003.md`
- best confirmed run: `run-2026-03-13-005`

Additional challenger testing on 2026-03-13 did not displace the baseline.

Rejected challengers:

- `gemma3:4b` — `run-2026-03-13-016`
- `qwen2.5:7b` — `run-2026-03-13-017`
- `gemma3:12b` — `run-2026-03-13-018`
- prompt challenger `lore-safe-proofread-004` with `qwen2.5:14b` — `run-2026-03-13-019`
- `llama3.1:8b` — `run-2026-03-13-020`
- `mistral:7b-instruct` — `run-2026-03-13-021`

## Reason
This combination is still the strongest confirmed result so far.

Run `run-2026-03-13-005` remains the quality anchor for the current baseline decision.

Later runs `run-2026-03-13-014` and `run-2026-03-13-015` verified that the same baseline combination still executes successfully through the wrapper-driven workflow, but they do not by themselves replace the best confirmed run.

Subsequent challenger testing also failed to produce a better accepted result.

This baseline returns clean output without reasoning leakage or commentary.
It preserves protected terms and literary tone better than the prior model baseline.
It still makes occasional minor phrasing drift, but overall it is acceptable as the current baseline.

## Next Improvement Target
Reduce the remaining phrasing drift without harming compliance.
