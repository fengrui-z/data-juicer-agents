---
name: plan
description: Build execution plan using atomic planning tools
when:
  - user_intent contains ["process", "clean", "filter", "deduplicate", "plan", "处理", "清洗", "过滤", "去重"]
  - no plan.yaml exists in context
next: apply.md
---

# Plan Skill

Build a Data-Juicer execution plan using atomic tools.

## Tool Chain

```
inspect_dataset
    -> retrieve_operators
    -> build_dataset_spec
    -> build_process_spec
    -> build_system_spec
    -> assemble_plan
    -> plan_validate
    -> plan_save
```

---

## Step 1: Inspect Dataset

Analyze the input dataset to identify modality, keys, and statistics.

**Tool:** `inspect_dataset`

```json
{
  "dataset_path": "./input.jsonl",
  "sample_size": 20
}
```

**Returns:** `dataset_profile` with modality, text_keys, image_key, sample stats.

---

## Step 2: Retrieve Operators

Find matching operators for the user's intent.

**Tool:** `retrieve_operators`

```json
{
  "intent": "deduplicate and filter short text",
  "top_k": 10,
  "mode": "auto",
  "dataset_path": "./input.jsonl"
}
```

**Returns:** List of candidate operators with names, descriptions, scores.

---

## Step 3: Build Dataset Spec

Create the dataset specification from inspection results.

**Tool:** `build_dataset_spec`

```json
{
  "intent": "deduplicate and filter short text",
  "dataset_path": "./input.jsonl",
  "export_path": "./output.jsonl",
  "dataset_profile": { /* from inspect_dataset */ }
}
```

**Optional overrides:**
- `modality_hint`: Force modality (text, image, video, audio)
- `text_keys_hint`: Override detected text keys
- `image_key_hint`: Override detected image key

**Returns:** `dataset_spec` object.

---

## Step 4: Build Process Spec

Define the operator sequence with parameters.

**Tool:** `build_process_spec`

```json
{
  "operators": [
    {
      "name": "whitespace_normalization_mapper",
      "params": {}
    },
    {
      "name": "text_length_filter",
      "params": {"min_len": 50, "max_len": 5000}
    },
    {
      "name": "document_deduplicator",
      "params": {"lowercase": false}
    }
  ]
}
```

**Returns:** `process_spec` object.

---

## Step 5: Build System Spec

Configure system-level settings (custom operators, parallelism).

**Tool:** `build_system_spec`

```json
{
  "custom_operator_paths": ["./custom_ops"]
}
```

**Returns:** `system_spec` object.

---

## Step 6: Assemble Plan

Combine all specs into a complete plan.

**Tool:** `assemble_plan`

```json
{
  "intent": "deduplicate and filter short text",
  "dataset_spec": { /* from build_dataset_spec */ },
  "process_spec": { /* from build_process_spec */ },
  "system_spec": { /* from build_system_spec */ },
  "approval_required": true
}
```

**Returns:** `plan_payload` object.

---

## Step 7: Validate Plan

Verify the plan is structurally valid.

**Tool:** `plan_validate`

```json
{
  "plan_payload": { /* from assemble_plan */ }
}
```

**Returns:** Validation result with any warnings.

---

## Step 8: Save Plan

Persist the plan to disk.

**Tool:** `plan_save`

```json
{
  "plan_payload": { /* from assemble_plan */ },
  "output_path": "./plans/my_plan.yaml",
  "overwrite": false
}
```

**Returns:** Saved file path.

---

## Validation Tools (Optional)

Use these to validate individual specs before assembly:

| Tool | Purpose |
|------|---------|
| `validate_dataset_spec` | Check dataset spec validity |
| `validate_process_spec` | Check process spec validity |
| `validate_system_spec` | Check system spec validity |

---

## Common Operator Patterns

| Goal | Operators |
|------|-----------|
| Text cleaning | `whitespace_normalization_mapper`, `punctuation_normalization_mapper` |
| Deduplication | `document_deduplicator`, `image_deduplicator` |
| Length filter | `text_length_filter` |
| Language filter | `language_id_score_filter` |
| Quality filter | `perplexity_filter`, `special_characters_filter` |

---

## Next Step

After `plan_save`, execute with [apply.md](apply.md):

```json
{"plan_path": "./plans/my_plan.yaml", "confirm": true}
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| inspect_dataset fails | Check dataset path and JSONL format |
| No operators found | Try broader intent or check spelling |
| Validation fails | Review operator names and params |

See [debug.md](debug.md) for detailed troubleshooting.
