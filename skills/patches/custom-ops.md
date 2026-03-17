---
name: custom-ops
description: Custom operator path and resolution issues
when:
  - error contains ["operator not found", "custom", "not registered", "unknown operator"]
---

# Patch: Custom Operator Issues

## Problem

Custom operator not found during plan validation or execution.

## Symptoms

- "Operator 'xxx' not found"
- "Unknown operator"
- Plan validation fails

---

## Solution

### Step 1: Verify operator file exists

```json
{"command": "ls -la ./custom_ops/", "timeout": 10}
```

### Step 2: Check operator content

```json
{"file_path": "./custom_ops/my_operator.py"}
```

### Step 3: Include in build_system_spec

```json
{
  "custom_operator_paths": ["./custom_ops"]
}
```

---

## Naming Convention

| Type | File Pattern | Class Pattern |
|------|--------------|---------------|
| Filter | `*_filter.py` | `*Filter` |
| Mapper | `*_mapper.py` | `*Mapper` |

### Examples

```
File: sentiment_threshold_filter.py
Class: SentimentThresholdFilter
Usage in build_process_spec: "sentiment_threshold_filter"
```

---

## Correct Workflow

```
// 1. Generate operator
develop_operator(
  intent="...",
  operator_name="my_filter",
  output_dir="./custom_ops",
  operator_type="filter"
)

// 2. Include path in system spec
build_system_spec(
  custom_operator_paths=["./custom_ops"]
)

// 3. Use operator in process spec
build_process_spec(
  operators=[
    {"name": "my_filter", "params": {"threshold": 0.5}}
  ]
)
```

---

## Multiple Custom Directories

```json
{
  "custom_operator_paths": ["./custom_ops", "./more_ops"]
}
```

---

## Operator Structure

Minimum required structure:

```python
from data_juicer.ops.filter import Filter

class MyCustomFilter(Filter):
    def __init__(self, threshold=0.5, **kwargs):
        super().__init__(**kwargs)
        self.threshold = threshold
    
    def compute(self, sample):
        return sample.get('score', 0) > self.threshold
```

---

## Path Resolution

Paths can be:
- **Relative**: `./custom_ops` (relative to working directory)
- **Absolute**: `/full/path/to/custom_ops`

Check current directory:

```json
{"command": "pwd", "timeout": 10}
```

---

## Regenerate Operator

If issues persist:

```json
{
  "intent": "my filter logic",
  "operator_name": "my_filter",
  "output_dir": "./custom_ops",
  "operator_type": "filter",
  "smoke_check": true
}
```

The `smoke_check: true` validates the generated operator.
