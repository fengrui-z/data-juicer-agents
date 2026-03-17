---
name: dev
description: Generate custom operator scaffold using develop_operator tool
when:
  - user_intent contains ["custom", "new operator", "develop", "自定义", "新算子", "开发"]
  - no built-in operator matches user requirement
next: plan.md
---

# Dev Skill

Generate custom operator scaffold using the `develop_operator` tool.

## Tool

**Tool:** `develop_operator`

```json
{
  "intent": "Filter samples by sentiment score threshold",
  "operator_name": "sentiment_threshold_filter",
  "output_dir": "./custom_ops",
  "operator_type": "filter",
  "from_retrieve": "",
  "smoke_check": true
}
```

---

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `intent` | string | Required | Description of what the operator should do |
| `operator_name` | string | "" | Operator name in snake_case |
| `output_dir` | string | "" | Output directory for generated files |
| `operator_type` | string | "" | Type: `mapper` or `filter` |
| `from_retrieve` | string | "" | Path to retrieve results for context |
| `smoke_check` | boolean | false | Run validation after generation |

---

## Examples

### Create a filter operator

```json
{
  "intent": "Filter samples by sentiment score above threshold",
  "operator_name": "sentiment_threshold_filter",
  "output_dir": "./custom_ops",
  "operator_type": "filter",
  "smoke_check": true
}
```

### Create a mapper operator

```json
{
  "intent": "Normalize Unicode characters to ASCII",
  "operator_name": "unicode_normalizer_mapper",
  "output_dir": "./custom_ops",
  "operator_type": "mapper"
}
```

### With context from retrieve

```json
// Step 1: Save retrieve results
retrieve_operators(intent="sentiment analysis", top_k=5)
// Save output to ./context.json

// Step 2: Generate with context
{
  "intent": "Filter by sentiment score",
  "operator_name": "sentiment_filter",
  "output_dir": "./custom_ops",
  "from_retrieve": "./context.json"
}
```

---

## Output Files

| File | Description |
|------|-------------|
| `<name>.py` | Operator implementation |
| `test_<name>.py` | Test scaffold |
| `<name>_SUMMARY.md` | Usage documentation |

---

## Operator Types

### Filter

- Returns `True` to keep sample, `False` to drop
- Use for: quality filtering, language filtering, length filtering

```python
class MyFilter(Filter):
    def compute(self, sample):
        return sample.get('score', 0) > self.threshold
```

### Mapper

- Transforms and returns modified sample
- Use for: text normalization, field extraction, format conversion

```python
class MyMapper(Mapper):
    def process(self, sample):
        sample['text'] = sample['text'].lower()
        return sample
```

---

## Naming Convention

| Type | File Pattern | Class Pattern |
|------|--------------|---------------|
| Filter | `*_filter.py` | `*Filter` |
| Mapper | `*_mapper.py` | `*Mapper` |

Example: `sentiment_threshold_filter` -> `SentimentThresholdFilter`

---

## Complete Workflow

```
// 1. (Optional) Find similar operators
retrieve_operators(intent="sentiment analysis")

// 2. Generate scaffold
develop_operator(
  intent="Filter by sentiment score",
  operator_name="sentiment_filter",
  output_dir="./custom_ops",
  operator_type="filter",
  smoke_check=true
)

// 3. Edit generated code if needed
view_text_file(file_path="./custom_ops/sentiment_filter.py")
write_text_file(file_path="./custom_ops/sentiment_filter.py", content="...")

// 4. Use in planning with custom_operator_paths
build_system_spec(custom_operator_paths=["./custom_ops"])
```

---

## Using Custom Operators in Plans

After creating a custom operator, include it in planning:

```json
// In build_system_spec
{"custom_operator_paths": ["./custom_ops"]}

// In build_process_spec
{"operators": [{"name": "sentiment_filter", "params": {"threshold": 0.5}}]}
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Smoke check fails | Review generated code, fix errors |
| Operator not found | Verify path in `custom_operator_paths` |
| Name mismatch | Use snake_case for `operator_name` |

See [debug.md](debug.md) for detailed troubleshooting.
