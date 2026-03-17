---
name: timeout
description: Execution timeout issues
when:
  - error contains ["timeout", "timed out", "exceeded", "deadline"]
  - process hangs or takes too long
---

# Patch: Timeout Handling

## Problem

Tool execution times out.

## Symptoms

- "Execution timed out"
- Tool hangs without response
- "Deadline exceeded"

---

## Solution

### apply_recipe timeout

Increase the `timeout` parameter:

```json
{
  "plan_path": "./plans/plan_xxx.yaml",
  "timeout": 1800,
  "confirm": true
}
```

### execute_shell_command timeout

```json
{
  "command": "your_command",
  "timeout": 300
}
```

### execute_python_code timeout

```json
{
  "code": "your_code",
  "timeout": 300
}
```

---

## Timeout Guidelines

| Dataset Size | Recommended Timeout |
|--------------|---------------------|
| < 10K samples | 300 (default) |
| 10K - 100K | 900 (15 min) |
| 100K - 1M | 1800 (30 min) |
| > 1M | 3600+ (1+ hour) |

---

## Slow Operations

Operations that commonly need longer timeouts:

| Tool | Default | Notes |
|------|---------|-------|
| `apply_recipe` | 300 | Increase for large datasets |
| `execute_shell_command` | 120 | Increase for long commands |
| `execute_python_code` | 120 | Increase for complex code |
| `retrieve_operators` (llm mode) | - | May be slow with LLM |

---

## Reduce Processing Time

### Smaller dataset for testing

```json
// Use inspect_dataset with smaller sample
{"dataset_path": "./input.jsonl", "sample_size": 10}
```

### Dry run first

```json
// Validate without execution
{"plan_path": "./plan.yaml", "dry_run": true}
```

---

## Check Progress

For long-running operations, check logs:

```json
{"file_path": "./data/log/export_xxx.txt"}
```
