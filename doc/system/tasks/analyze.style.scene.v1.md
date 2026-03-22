# Task Contract: analyze.style.scene.v1

Date: 2026-03-21

## Identity

- **task_id:** `analyze.style.scene.v1`
- **contract_version:** `v1`
- **family:** `analysis`
- **task_type:** `style_analysis`
- **scope:** `scene`
- **trust_posture:** `candidate_only`
- **route_class:** `WORKHORSE_LOCAL`
- **strictness:** `STRUCTURED_ANALYSIS`
- **fallback_policy:** `degraded_allowed`

---

## Purpose

Accept a bounded single-scene text payload and return a structured advisory analysis artifact covering style dimensions: clarity, flow, voice_consistency, sentence_variety, and pacing.

This is the first live task contract for style analysis in NeuronForge.

---

## Non-goals for v1

This contract does not support:

- chapter-window or manuscript-scope execution
- automatic canonical writes
- silent prose fallback on schema failure
- promotion or lifecycle management
- user-supplied style goals or voice constraints (deferred to v2)

---

## V1 frozen dimension set

The following dimensions are frozen for v1 and are non-configurable:

| Dimension | Key | Range |
|-----------|-----|-------|
| Clarity | `clarity` | 0.0–1.0 |
| Flow | `flow` | 0.0–1.0 |
| Voice Consistency | `voice_consistency` | 0.0–1.0 |
| Sentence Variety | `sentence_variety` | 0.0–1.0 |
| Pacing | `pacing` | 0.0–1.0 |

---

## Request shape

```json
{
  "request_id": "<string>",
  "task_family": "analysis",
  "task_type": "style_analysis",
  "contract_version": "v1",
  "source_scope": "scene",
  "input_payload": {
    "scene_text": "<string>"
  },
  "desired_runtime_mode": "WORKHORSE_LOCAL",
  "output_strictness": "structured"
}
```

### Required request fields

| Field | Type | Constraint |
|-------|------|-----------|
| `request_id` | string | Non-empty |
| `task_family` | literal | Must be `"analysis"` |
| `task_type` | literal | Must be `"style_analysis"` |
| `contract_version` | literal | Must be `"v1"` |
| `source_scope` | literal | Must be `"scene"` |
| `input_payload.scene_text` | string | Non-empty |
| `desired_runtime_mode` | string | Default `"WORKHORSE_LOCAL"` |
| `output_strictness` | literal | Must be `"structured"` |

---

## Response envelope shape

On success:

```json
{
  "route_class": "WORKHORSE_LOCAL",
  "model_id": "<model-name>",
  "runtime_mode_used": "WORKHORSE_LOCAL",
  "provenance_class": "inferred_candidate",
  "schema_validation_status": "valid",
  "warnings": [],
  "output_payload": {
    "summary": "<string>",
    "overall_assessment": "<string>",
    "dimension_scores": {
      "clarity": 0.0,
      "flow": 0.0,
      "voice_consistency": 0.0,
      "sentence_variety": 0.0,
      "pacing": 0.0
    },
    "findings": [],
    "recommendations": [],
    "confidence": 0.0,
    "evidence_spans": []
  }
}
```

### Schema validation status values

| Value | Meaning |
|-------|---------|
| `valid` | All required fields present, scores valid, schema passes |
| `degraded` | Required fields present but confidence < 0.4, or evidence_spans empty, or some dimension scores missing (filled with 0.0) |
| `failed` | JSON parse failure, missing summary/overall_assessment/dimension_scores, or prose-only output |

---

## Fail response shape

On failed schema validation or parse failure:

```json
{
  "route_class": "WORKHORSE_LOCAL",
  "model_id": "<model-name>",
  "runtime_mode_used": "WORKHORSE_LOCAL",
  "provenance_class": "inferred_candidate",
  "schema_validation_status": "failed",
  "warnings": ["<reason>"],
  "output_payload": null
}
```

---

## Output payload schema

### StyleAnalysisOutputPayload

| Field | Type | Constraint |
|-------|------|-----------|
| `summary` | string | Non-empty |
| `overall_assessment` | string | Non-empty |
| `dimension_scores` | object | Keys: clarity, flow, voice_consistency, sentence_variety, pacing; values 0.0–1.0 |
| `findings` | array | List of StyleFinding objects |
| `recommendations` | array | List of StyleRecommendation objects |
| `confidence` | float | 0.0–1.0 |
| `evidence_spans` | array | List of EvidenceSpan objects |

### StyleFinding

| Field | Type | Constraint |
|-------|------|-----------|
| `type` | literal | `"strength"`, `"weakness"`, or `"observation"` |
| `label` | string | Non-empty |
| `detail` | string | Non-empty |

### StyleRecommendation

| Field | Type | Constraint |
|-------|------|-----------|
| `priority` | literal | `"high"`, `"medium"`, or `"low"` |
| `label` | string | Non-empty |
| `detail` | string | Non-empty |

### EvidenceSpan

| Field | Type | Constraint |
|-------|------|-----------|
| `start` | int | >= 0 |
| `end` | int | > start |
| `reason` | string | Non-empty |

---

## Normalization rules

### Success
- All required fields present
- All dimension scores parseable as float, clampable to 0.0–1.0
- summary and overall_assessment non-empty strings
- schema validates

### Degraded
Required fields present AND at least one of:
- `evidence_spans` is empty
- `confidence` < 0.4
- One or more dimension scores missing (filled with 0.0, warned)

### Failed
Any of:
- JSON cannot be parsed from model output
- `summary` or `overall_assessment` missing or empty
- `dimension_scores` missing entirely
- Output is prose-only with no extractable JSON object

---

## Execution path

```
POST /api/v1/authorforge/style-analysis
  → validate request envelope (fail closed on malformed input)
  → build model input (system prompt + scene text)
  → run WORKHORSE_LOCAL model via ollama
  → extract JSON from raw output (strip think blocks, markdown fences)
  → normalize output against schema
  → [failed] → response with schema_validation_status: "failed", output_payload: null
  → [degraded] → response with schema_validation_status: "degraded", warnings
  → [valid] → response with schema_validation_status: "valid"
  → return structured response envelope
```

---

## Files

| File | Purpose |
|------|---------|
| `prompts/style-analysis-scene-v1.md` | Model system prompt for this task |
| `scripts/style_analysis/models.py` | Pydantic request/response models |
| `scripts/style_analysis/normalizer.py` | Output normalization and validation |
| `scripts/style_analysis/adapter.py` | Ollama model invocation adapter |
| `scripts/style_analysis/app.py` | FastAPI app with route |
| `scripts/run-style-analysis.sh` | CLI executor |
| `tests/test-style-analysis.py` | Contract-level tests |

---

## Governance notes

- This contract is `v1`. Changes to required fields, dimensions, or validation rules require a version bump.
- Consumer code must bind to this contract, not to prompt internals.
- The prompt is hidden behind the contract boundary.
- Model selection is separate from the contract and governed by the route class policy.
- This contract is advisory only. Output is `inferred_candidate`, never `authority_persisted`.
