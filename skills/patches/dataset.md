---
name: dataset
description: Dataset format and parsing issues
when:
  - error contains ["JSONL", "JSON", "parse", "invalid", "field", "text", "format"]
---

# Patch: Dataset Format Issues

## Problem

Dataset parsing fails in `inspect_dataset` or during execution.

## Symptoms

- "Invalid JSONL format"
- "Field 'text' not found"
- "JSON parse error"

---

## Solution

### Step 1: Validate JSONL format

```json
{"command": "head -1 ./input.jsonl | python -m json.tool", "timeout": 10}
```

### Step 2: Check file structure

```json
{"command": "head -5 ./input.jsonl", "timeout": 10}
```

### Step 3: Verify field names

```json
{"command": "head -1 ./input.jsonl | python -c \"import json,sys; print(json.load(sys.stdin).keys())\"", "timeout": 10}
```

---

## Valid JSONL Format

**Correct:**
```jsonl
{"text": "Sample 1", "id": 1}
{"text": "Sample 2", "id": 2}
```

**Wrong - array format:**
```json
[{"text": "Sample 1"}, {"text": "Sample 2"}]
```

**Wrong - invalid JSON:**
```
{text: "Missing quotes"}
```

---

## Using inspect_dataset

```json
{
  "dataset_path": "./input.jsonl",
  "sample_size": 5
}
```

This will report detected fields and any parsing issues.

---

## Common Fixes

### Convert array to JSONL

```json
{"command": "python -c \"import json; [print(json.dumps(x)) for x in json.load(open('input.json'))]\" > input.jsonl", "timeout": 60}
```

### Remove BOM

```json
{"command": "sed -i '' '1s/^\\xEF\\xBB\\xBF//' input.jsonl", "timeout": 10}
```

### Fix encoding

```json
{"command": "iconv -f ISO-8859-1 -t UTF-8 input.jsonl > input_utf8.jsonl", "timeout": 60}
```

---

## Custom Text Field

If your dataset uses a different field name (e.g., `content` instead of `text`):

Use hints in `build_dataset_spec`:

```json
{
  "intent": "...",
  "dataset_path": "./input.jsonl",
  "export_path": "./output.jsonl",
  "dataset_profile": { /* from inspect_dataset */ },
  "text_keys_hint": ["content"]
}
```

---

## Multimodal Datasets

For image datasets, verify image paths exist:

```json
{"command": "head -1 ./input.jsonl | python -c \"import json,sys,os; d=json.load(sys.stdin); print(os.path.exists(d.get('image','')))\"", "timeout": 10}
```

Use image key hint if needed:

```json
{
  "dataset_profile": { /* ... */ },
  "image_key_hint": "image_path"
}
```

---

## View File Content

```json
{"file_path": "./input.jsonl", "ranges": [1, 10]}
```
