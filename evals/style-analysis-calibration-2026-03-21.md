# Style Analysis Calibration
Date: 2026-03-21

## Lane
- lane_id: analyze-style-scene-v1
- lane_name: Style Analysis — Scene (v1)
- lane_type: style_analysis

## Purpose
This is the first structured evaluation run for the `analyze.style.scene.v1` capability. The goal is to measure schema reliability, assess whether style findings are plausible for scenes with known characteristics, and produce a baseline judgment sufficient to promote the lane from `implementing` to `evaluating`.

This evaluation is not a full adoption review. It establishes that the capability produces well-formed, useful output across a range of scene types and that the evaluation infrastructure works end-to-end.

---

## Model and configuration
- model: qwen2.5:14b
- prompt profile: style-analysis-scene-v1
- executor: scripts/run-style-analysis.sh
- route class: WORKHORSE_LOCAL
- runtime: Ollama (local)
- contract: analyze.style.scene.v1

---

## Evaluation set

Five scene files were created under `inputs/style-analysis-eval/`. Each was designed to test a distinct style profile:

| Scene file | Design intent | Expected signals |
|------------|--------------|-----------------|
| scene-01-clean.md | Well-written prose. Good clarity, varied sentences, consistent voice. | High scores, few weaknesses, mostly strengths. |
| scene-02-dense.md | Overwritten / purple prose. Excessive adjectives, long exhausting sentences, poor pacing. | Low pacing, low sentence_variety, multiple weaknesses. |
| scene-03-flat.md | Flat / monotone. Short repetitive sentences, no variation, minimal voice. | Low sentence_variety, low flow. |
| scene-04-voice-drift.md | POV voice shift mid-scene (close-third to first-person intrusion). | Low voice_consistency. |
| scene-05-dialogue-heavy.md | Dialogue-dominated scene with thin action/description balance. | Findings on descriptive density, pacing observations. |

---

## Runs executed

| Run ID | Scene | Model | schema_validation_status |
|--------|-------|-------|--------------------------|
| run-2026-03-21-001 | scene-01-clean.md | qwen2.5:14b | valid |
| run-2026-03-21-002 | scene-02-dense.md | qwen2.5:14b | valid |
| run-2026-03-21-003 | scene-03-flat.md | qwen2.5:14b | valid |
| run-2026-03-21-004 | scene-04-voice-drift.md | qwen2.5:14b | valid |
| run-2026-03-21-005 | scene-05-dialogue-heavy.md | qwen2.5:14b | valid |

All 5 runs returned `schema_validation_status: valid`. No runs degraded or failed. Warnings list was empty on all runs.

**Note:** The shell executor (`run-style-analysis.sh`) has a path resolution bug in its embedded normalizer invocation that causes it to log `failed` for the envelope written by the script. Raw model output is correct. The actual normalization was re-run using the correct `sys.path` and all 5 envelopes are accurate in `evals/style-analysis-eval-2026-03-21/raw/scene-0N-response.json`. The shell script path bug should be fixed in a follow-up.

---

## Per-scene results

### scene-01-clean.md (run-2026-03-21-001)
**Design intent:** Clean, well-written scene. Expect high scores, few weaknesses.

**Dimension scores:**
- clarity: 0.90
- flow: 0.85
- voice_consistency: 0.75
- sentence_variety: 0.95
- pacing: 0.80

**Confidence:** 0.90
**Findings:** 1 strength (rich sensory details), 1 weakness (voice inconsistency noted as minor narrative shift)
**Recommendations:** 2 (medium: maintain consistent voice; low: vary sentence length)
**Evidence spans:** 2

**Operator assessment:** Scores are plausible and directionally correct. The clean scene received the highest sentence_variety score across all five inputs. The voice_consistency score of 0.75 is slightly lower than expected for a well-written scene — the model identified a subtle narrative distance shift that is genuinely present in the scene. This is a reasonable judgment, not a false positive. Output is useful.

---

### scene-02-dense.md (run-2026-03-21-002)
**Design intent:** Overwritten/purple prose. Expect low pacing, low sentence_variety, multiple weaknesses.

**Dimension scores:**
- clarity: 0.80
- flow: 0.75
- voice_consistency: 1.00
- sentence_variety: 0.60
- pacing: 0.60

**Confidence:** 0.90
**Findings:** 1 strength (effective imagery), 1 weakness (overuse of adjectives / cumbersome sentences), 1 observation (consistent tone)
**Recommendations:** 2 (medium: vary sentence structure; low: trim redundant adjectives)
**Evidence spans:** 2

**Operator assessment:** Pacing (0.60) and sentence_variety (0.60) correctly score low, matching the design intent. The weakness finding on adjective overuse is accurate and well-targeted. voice_consistency scoring at 1.00 is a notable outlier — the purple prose is tonally consistent, which the model correctly identified, but the score seems high given the qualitative issues. This may indicate the model interprets voice_consistency narrowly as tonal coherence rather than overall prose quality. Findings and recommendations are actionable and correct. Output is useful.

---

### scene-03-flat.md (run-2026-03-21-003)
**Design intent:** Flat/monotone scene with short repetitive sentences. Expect low sentence_variety, low flow.

**Dimension scores:**
- clarity: 0.90
- flow: 0.60
- voice_consistency: 0.50
- sentence_variety: 0.30
- pacing: 0.70

**Confidence:** 0.80
**Findings:** 1 strength (clear description), 1 weakness (lack of sentence variety — repetitive), 1 observation (minimal voice)
**Recommendations:** 2 (high: increase sentence variety; medium: develop voice)
**Evidence spans:** 2

**Operator assessment:** sentence_variety (0.30) is the lowest score across the entire eval set, correctly identifying the flat repetitive structure. flow (0.60) also correctly scores low. The high-priority recommendation on sentence variety is appropriate. voice_consistency scoring low (0.50) is reasonable — the flat monotone style has no discernible authorial voice. The model correctly diagnosed the dominant problem. Output is useful.

---

### scene-04-voice-drift.md (run-2026-03-21-004)
**Design intent:** Mid-scene POV voice shift (close-third to embedded first-person intrusion). Expect low voice_consistency.

**Dimension scores:**
- clarity: 0.90
- flow: 0.70
- voice_consistency: 0.85
- sentence_variety: 0.65
- pacing: 0.70

**Confidence:** 0.90
**Findings:** 1 strength (strong descriptive clarity), 1 weakness (uneven pacing — shift from action to reflective narration)
**Recommendations:** 2 (medium: reduce introspection for smoother flow; high: enhance sentence variety)
**Evidence spans:** 2

**Operator assessment:** This is the most notable calibration gap in the eval set. The scene contains a deliberate mid-scene shift from close-third POV to first-person introspective narration — a clear voice consistency failure. The model identified the intrusion (correctly noting the reflective narration disrupts the action-driven flow) and located it correctly in the evidence spans (characters 145–238), but classified it primarily as a flow/pacing issue rather than a voice_consistency issue. voice_consistency scored 0.85, which is high given the explicit POV shift. This represents a partial surface detection: the problem was found and located but miscategorized at the dimension level. This is a meaningful calibration note: the model's voice_consistency dimension may be interpreted as tonal register consistency rather than POV fidelity.

---

### scene-05-dialogue-heavy.md (run-2026-03-21-005)
**Design intent:** Dialogue-dominated scene with thin action/description balance. Expect findings on descriptive density and pacing.

**Dimension scores:**
- clarity: 0.80
- flow: 0.75
- voice_consistency: 0.90
- sentence_variety: 0.60
- pacing: 0.80

**Confidence:** 0.85
**Findings:** 1 strength (clear dialogue), 1 weakness (lack of sentence variety / simple sentences)
**Recommendations:** 2 (medium: increase sentence complexity; low: incorporate descriptive elements)
**Evidence spans:** 2

**Operator assessment:** The low recommendation on descriptive elements correctly identifies the thin action/description balance. sentence_variety scoring low (0.60) is accurate given the staccato dialogue-only rhythm. Pacing scoring 0.80 suggests the model read the dialogue's brisk momentum as acceptable pacing, which is arguable — the scene does move, even if it lacks descriptive grounding. The findings are actionable and the output is useful. No false positives observed.

---

## Metric derivation

### schema_reliability
Measured as: fraction of runs returning `schema_validation_status: valid` with all 5 dimension scores present, non-empty findings and recommendations, non-empty evidence_spans, and non-empty summary/overall_assessment.

5 of 5 runs: valid
**schema_reliability = 1.00**

### false_positive_rate
Interpreted for style analysis as: rate of findings that are clearly incorrect, unfounded, or inapplicable to the scene.

Operator review across all 5 runs identified zero findings that were clearly incorrect or inapplicable. All weaknesses identified were present in the scenes. The voice_consistency scoring on scene-04 was a miscategorization rather than a false positive (the issue was real, just attributed to the wrong dimension).

**false_positive_rate = 0.00** (operator judgment, 5-scene set)

### surface_detection_rate
Interpreted for style analysis as: rate of real, intentionally-embedded style issues correctly identified by the capability.

Embedded issues and detection outcomes:
- scene-01: No major embedded issues. Model identified minor voice distance shift — plausible. (not scored as detection miss)
- scene-02: Adjective overuse and poor pacing. Both detected. **Hit.**
- scene-03: Sentence variety failure. Correctly detected as dominant issue with high-priority recommendation. **Hit.**
- scene-04: POV voice shift. Issue was found and located correctly but miscategorized (flow vs. voice_consistency dimension). **Partial hit.** Counted as detected.
- scene-05: Thin descriptive balance. Correctly identified in recommendation. **Hit.**

4 of 4 targeted issues detected (scene-04 counted as detected despite miscategorization).

**surface_detection_rate = 1.00** (operator judgment, 5-scene set — note: small sample, scene-04 miscategorization is a calibration concern not reflected in this binary metric)

---

## Judgment summary

The `analyze.style.scene.v1` capability with `qwen2.5:14b` produces valid, parseable, schema-conformant output on all 5 eval scenes. Schema reliability is 1.00 across this set. Findings are generally accurate and non-spurious. The capability correctly diagnoses dominant style problems (sentence variety failure, adjective overuse, thin description) in the scenes designed to exhibit those problems.

**Key calibration concern:** The `voice_consistency` dimension may track tonal register consistency rather than POV fidelity. Scene-04 exposed this: the model correctly found the intrusive first-person narration block and cited its location, but attributed it to flow/pacing rather than voice_consistency. Operators using this capability to detect POV drift should be aware that findings (especially evidence spans) may be more reliable than the voice_consistency score alone.

**Confidence:** Moderate. The eval set is 5 scenes and judgment is operator-derived. Metrics are not benchmark-derived. This is sufficient to promote from `implementing` to `evaluating` but not sufficient for baseline adoption.

---

## Next required decision

Review the voice_consistency miscategorization pattern more closely. Run at least 3 additional scenes specifically designed as POV-shift tests to determine whether scene-04's result was a model edge case or a systematic dimension interpretation gap. If systematic, consider whether the prompt should clarify what voice_consistency measures. Once additional POV-shift tests are complete, assess whether the lane is ready for `candidate_baseline` promotion.

---

## Challenger run: qwen3:14b (2026-03-21)

Model: qwen3:14b
Date: 2026-03-21
All 5 runs returned `schema_validation_status: valid`. No warnings on any run. Evidence spans count: 3 per scene (vs 2 per scene for qwen2.5:14b baseline).

### Dimension score comparison

| Scene | Dimension | qwen2.5:14b | qwen3:14b | Delta |
|-------|-----------|-------------|-----------|-------|
| 01-clean | clarity | 0.90 | 0.95 | +0.05 |
| 01-clean | flow | 0.85 | 0.92 | +0.07 |
| 01-clean | voice_consistency | 0.75 | 0.98 | +0.23 |
| 01-clean | sentence_variety | 0.95 | 0.88 | -0.07 |
| 01-clean | pacing | 0.80 | 0.85 | +0.05 |
| 01-clean | confidence | 0.90 | 0.90 | 0.00 |
| 02-dense | clarity | 0.80 | 0.70 | -0.10 |
| 02-dense | flow | 0.75 | 0.85 | +0.10 |
| 02-dense | voice_consistency | 1.00 | 0.95 | -0.05 |
| 02-dense | sentence_variety | 0.60 | 0.80 | +0.20 |
| 02-dense | pacing | 0.60 | 0.70 | +0.10 |
| 02-dense | confidence | 0.90 | 0.80 | -0.10 |
| 03-flat | clarity | 0.90 | 0.95 | +0.05 |
| 03-flat | flow | 0.60 | 0.75 | +0.15 |
| 03-flat | voice_consistency | 0.50 | 0.90 | +0.40 |
| 03-flat | sentence_variety | 0.30 | 0.60 | +0.30 |
| 03-flat | pacing | 0.70 | 0.70 | 0.00 |
| 03-flat | confidence | 0.80 | 0.85 | +0.05 |
| 04-voice-drift | clarity | 0.90 | 0.90 | 0.00 |
| 04-voice-drift | flow | 0.70 | 0.85 | +0.15 |
| 04-voice-drift | voice_consistency | 0.85 | 0.90 | +0.05 |
| 04-voice-drift | sentence_variety | 0.65 | 0.80 | +0.15 |
| 04-voice-drift | pacing | 0.70 | 0.85 | +0.15 |
| 04-voice-drift | confidence | 0.90 | 0.90 | 0.00 |
| 05-dialogue-heavy | clarity | 0.80 | 0.90 | +0.10 |
| 05-dialogue-heavy | flow | 0.75 | 0.85 | +0.10 |
| 05-dialogue-heavy | voice_consistency | 0.90 | 0.90 | 0.00 |
| 05-dialogue-heavy | sentence_variety | 0.60 | 0.75 | +0.15 |
| 05-dialogue-heavy | pacing | 0.80 | 0.85 | +0.05 |
| 05-dialogue-heavy | confidence | 0.85 | 0.85 | 0.00 |

### Schema reliability

qwen3:14b: 5/5 valid. No warnings. Evidence spans: 3 per scene consistently (vs 2 per scene for qwen2.5:14b). Schema reliability = 1.00. Matches baseline.

### Evidence span quality

qwen3:14b produces 3 evidence spans per scene vs 2 for qwen2.5:14b. Span citations are specific and well-reasoned. On scene-04, qwen3 spans include a direct citation of the first-person intrusion text (`"I have always been a careful person..."`) at chars 142–163 and an explicit "Perspective shift" label on the third-person-to-first-person transition. This is meaningfully more informative than the qwen2.5:14b span which cited the reflective narration block (145–238) under the label "flow disruption" only.

### voice_consistency on scene-04

The key calibration concern from the baseline run was that qwen2.5:14b scored voice_consistency 0.85 on scene-04 (designed as a POV-shift scene) despite correctly locating the first-person intrusion in the evidence spans. The model attributed the issue to pacing/flow rather than voice_consistency.

**qwen3:14b result:**

- voice_consistency score: **0.90** — higher than the baseline 0.85. The POV shift miscategorization is not resolved; it is slightly worse at the dimension score level.
- However, qwen3 produced an explicit "Perspective shifts" observation finding with the detail: "The transition from third-person narration to first-person introspection is handled but may require tighter integration for seamless immersion." This correctly names the POV shift as a finding.
- Evidence span at 142–163 directly quotes the first-person intrusion (`I have always been a careful person...`) as evidence, and span at 332–354 labels the return to third-person as a "Perspective shift."
- The low-priority recommendation "Refine perspective transitions" explicitly mentions smoothing the shift between third-person narration and first-person introspection.

**Assessment:** qwen3:14b identifies the POV shift more explicitly in its findings and evidence spans than qwen2.5:14b, which only mentioned "reflective narration disrupting flow." However, neither model correctly lowers voice_consistency to reflect the POV drift; both produce high scores (0.85 and 0.90) on a scene specifically designed to exhibit voice inconsistency. The systematic pattern is confirmed: the voice_consistency dimension score does not reliably penalize POV drift under the current prompt. This is a prompt-level calibration gap, not a model capability gap — both models can locate and label the shift in findings and evidence spans but do not translate it to a lower voice_consistency score.

qwen3:14b is marginally better at *labeling* the issue explicitly (observation finding vs. no voice label in qwen2.5), but the score-level miscategorization is identical in character (high voice_consistency score despite POV shift).

### General scoring behavior differences

qwen3:14b scores are systematically higher across most dimensions and most scenes. Notable patterns:
- Sentence_variety: qwen3 scored this dimension considerably higher than qwen2.5 on scenes 02-dense (+0.20), 03-flat (+0.30), 04-voice-drift (+0.15), and 05-dialogue-heavy (+0.15). This reduces the discriminative signal on scenes designed to exhibit low sentence variety.
- On scene-03-flat, qwen3 scored sentence_variety 0.60 while qwen2.5 scored 0.30. Scene-03 was specifically designed to exhibit extreme sentence flatness. The qwen2.5:14b score of 0.30 is the more diagnostically correct signal; qwen3:14b undershoots the severity.
- voice_consistency is inflated on scene-03-flat (qwen3: 0.90 vs qwen2.5: 0.50). qwen2.5:14b's lower score is more accurate for a scene with minimal authorial voice.
- On scene-02-dense, qwen3 correctly scores clarity lower (0.70 vs 0.80), which is directionally better. Sentence_variety is over-scored (0.80 vs 0.60); qwen2.5:14b's 0.60 is more accurate for the dense prose scene.

### Judgment

**Neither model resolves the voice_consistency miscategorization on scene-04.** The systematic issue is confirmed as a prompt-level calibration gap: the prompt does not explicitly instruct the model to penalize POV-perspective drift in the voice_consistency score. Both models detect the POV shift in evidence spans but score voice_consistency high.

**qwen3:14b does not outperform qwen2.5:14b on the key calibration concern.** On the dimension score level, qwen3 scores voice_consistency *higher* (0.90 vs 0.85) on scene-04. Its advantage is in richer finding labels and better evidence span specificity.

**qwen3:14b shows score inflation across most dimensions**, reducing discriminative power on the scenes designed to test low-scoring behavior (scene-02-dense, scene-03-flat). qwen2.5:14b's more conservative scoring profile is a better fit for diagnostic use cases where low scores are meaningful signals.

**qwen3:14b is not the preferred model candidate** based on this evaluation. qwen2.5:14b remains the baseline. The primary next action should be prompt calibration to define voice_consistency as including POV fidelity, tested against both models, rather than a model switch.

**next_required_decision update:** The POV-shift miscategorization is confirmed as systematic across both models and is a prompt-level gap. The prompt should be updated to define voice_consistency to include POV fidelity. Both models should be re-tested against the updated prompt on scene-04 and at least 2 additional POV-shift scenes before any candidate_baseline decision.
