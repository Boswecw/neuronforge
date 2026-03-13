# Run Registry

## Fields
- run id
- date
- model
- prompt file
- input file
- output file
- task
- notes

## Entries
- run id: run-2026-03-13-001
  date: 2026-03-13
  model: deepseek-r1:7b
  prompt file: prompts/proofread-basic-001.md
  input file: inputs/test-proofread-001.md
  output file: outputs/deepseek-r1-7b-proofread-001.md
  task: baseline proofreading test
  notes: produced visible reasoning and violated return-only-text rule; made some correct edits but also introduced a meaning shift
- run id: run-2026-03-13-002
  date: 2026-03-13
  model: deepseek-r1:7b
  prompt file: prompts/lore-safe-proofread-001.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/deepseek-r1-7b-lore-safe-001.md
  task: lore-safe proofreading baseline
  notes: failed hard; exposed reasoning, added commentary, violated return-only-text rule, altered imagery and meaning
- run id: run-2026-03-13-003
  date: 2026-03-13
  model: deepseek-r1:7b
  prompt file: prompts/lore-safe-proofread-002.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/deepseek-r1-7b-lore-safe-002.md
  task: lore-safe proofreading prompt revision test
  notes: still failed; reasoning leakage remained, commentary remained, output format failed, and proofreading quality remained unreliable
- run id: run-2026-03-13-004
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-002.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-001.md
  task: lore-safe proofreading comparison baseline
  notes: strong improvement over deepseek-r1:7b; no reasoning leakage or commentary; output format passed; still made some non-minimal wording changes; speed somewhat slow
- run id: run-2026-03-13-005
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-002.md
  task: lore-safe proofreading minimal-edit revision test
  notes: best result so far; clean output, no reasoning leakage, protected terms preserved, unnecessary rewrites reduced; still changed "sat badly in him" to "sat badly with him"
- run id: run-2026-03-13-006
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-003.md
  task: lore-safe proofreading repeatability check via script
  notes: script worked; output remained clean and compliant; slight wording drift remained between repeated runs, so behavior is usable but not perfectly deterministic
- run id: run-2026-03-13-007
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-004.md
  task: lore-safe proofreading review-workflow stability check
  notes: review helper script worked correctly; output matched prior run exactly; no diff against outputs/qwen2.5-14b-lore-safe-003.md

- run id: run-2026-03-13-008
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-004.md
  task: helper script logging test
  notes: test entry created by scripts/log-run.sh

- run id: run-2026-03-13-009
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-005.md
  task: wrapper script end-to-end test
  notes: proofread and log completed through wrapper script

- run id: run-2026-03-13-011
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-006.md
  task: wrapper auto-date test
  notes: date omitted intentionally; wrapper should fill with system date

- run id: run-2026-03-13-012
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-007.md
  task: wrapper auto-run-id test
  notes: run id and date omitted intentionally; wrapper should fill both

- run id: run-2026-03-13-013
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-test-001-run-2026-03-13-013.md
  task: wrapper auto-output test
  notes: run id, date, and output omitted intentionally; wrapper should fill all three

- run id: run-2026-03-13-014
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-test-001-run-2026-03-13-014.md
  task: proofread
  notes: live run verification

- run id: run-2026-03-13-015
  date: 2026-03-13
  model: qwen2.5:14b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/qwen2.5-14b-lore-safe-test-001-run-2026-03-13-015.md
  task: proofread
  notes: post-hardening live run

- run id: run-2026-03-13-016
  date: 2026-03-13
  model: gemma3:4b
  prompt file: prompts/lore-safe-proofread-003.md
  input file: inputs/lore-safe-test-001.md
  output file: outputs/gemma3-4b-lore-safe-test-001-run-2026-03-13-016.md
  task: lore-safe proofreading challenger test
  notes: stayed inside output contract and preserved terms and tone, but failed challenger review due to core grammar errors

