---
name: apply
description: Execute a saved plan using apply_recipe tool
when:
  - plan.yaml file exists
  - user_intent contains ["execute", "run", "apply", "执行", "运行"]
prev: plan.md
---

# Apply Skill

Execute a saved Data-Juicer plan using the `apply_recipe` tool.

## Tool

**Tool:** `apply_recipe`

```json
{
  "plan_path": "./plans/plan_xxx.yaml",
  "dry_run": false,
  "timeout": 300,
  "confirm": true
}
```

---

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `plan_path` | string | Required | Path to plan YAML file |
| `dry_run` | boolean | false | Validate only, don't execute |
| `timeout` | integer | 300 | Execution timeout in seconds (min: 1) |
| `confirm` | boolean | false | Explicit confirmation before execution |

---

## Examples

### Standard execution

```json
{
  "plan_path": "./plans/plan_abc123.yaml",
  "dry_run": false,
  "timeout": 300,
  "confirm": true
}
```

### Validate without execution

```json
{
  "plan_path": "./plans/plan_abc123.yaml",
  "dry_run": true
}
```

### Large dataset (extended timeout)

```json
{
  "plan_path": "./plans/plan_abc123.yaml",
  "timeout": 1800,
  "confirm": true
}
```

---

## Behavior

1. Loads plan YAML from `plan_path`
2. Writes recipe to `.djx/recipes/<plan_id>.yaml`
3. Executes `dj-process` (unless `dry_run: true`)
4. Returns execution status

---

## Return Value

```json
{
  "ok": true,
  "execution_id": "...",
  "status": "success",
  "recipe_path": ".djx/recipes/plan_xxx.yaml"
}
```

### Status Values

| Status | Meaning |
|--------|---------|
| `success` | Execution completed |
| `failed` | Execution error |
| `interrupted` | User cancelled or timeout |

---

## Workflow Integration

### Complete planning -> execution flow

```
inspect_dataset
    -> retrieve_operators
    -> build_dataset_spec
    -> build_process_spec
    -> build_system_spec
    -> assemble_plan
    -> plan_validate
    -> plan_save
    -> apply_recipe  <-- This tool
```

### Safe execution pattern

```
# Step 1: Validate
apply_recipe(plan_path="./plans/plan.yaml", dry_run=true)

# Step 2: Execute
apply_recipe(plan_path="./plans/plan.yaml", confirm=true)
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

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Plan file not found | Verify `plan_path` is correct |
| Timeout | Increase `timeout` parameter |
| Operator not found | Check custom operator paths in plan |

See [debug.md](debug.md) for detailed troubleshooting.
