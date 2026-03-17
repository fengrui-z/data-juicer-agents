---
name: debug
description: Troubleshoot tool execution errors
when:
  - tool execution failed
  - error message present
  - user_intent contains ["error", "failed", "not working", "出错", "失败", "排查", "问题"]
patches:
  - patches/api-keys.md
  - patches/timeout.md
  - patches/custom-ops.md
  - patches/dataset.md
---

# Debug Skill

Troubleshoot Data-Juicer tool execution errors.

## Quick Diagnosis

| Error Pattern | Likely Cause | Go To |
|---------------|--------------|-------|
| `API`, `key`, `unauthorized`, `401` | API configuration | [patches/api-keys.md](patches/api-keys.md) |
| `timeout`, `timed out` | Execution timeout | [patches/timeout.md](patches/timeout.md) |
| `operator not found`, `custom` | Custom operator path | [patches/custom-ops.md](patches/custom-ops.md) |
| `JSONL`, `parse`, `invalid`, `field` | Dataset format | [patches/dataset.md](patches/dataset.md) |

---

## Common Tool Errors

### inspect_dataset Errors

| Error | Cause | Solution |
|-------|-------|----------|
| File not found | Invalid path | Check `dataset_path` |
| Parse error | Invalid JSONL | Validate file format |

**Debug:**
```json
{
  "command": "head -1 ./input.jsonl | python -m json.tool"
}
```

---

### retrieve_operators Errors

| Error | Cause | Solution |
|-------|-------|----------|
| No candidates | Poor intent | Try broader search terms |
| API error | Missing key | Set `DASHSCOPE_API_KEY` |

**Debug:**
```json
{"intent": "list all operators", "top_k": 50, "mode": "vector"}
```

---

### build_*_spec Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Invalid profile | Bad inspect result | Re-run `inspect_dataset` |
| Unknown operator | Typo in name | Check `retrieve_operators` results |

---

### apply_recipe Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Plan not found | Wrong path | Verify `plan_path` |
| Timeout | Large dataset | Increase `timeout` |
| Operator error | Bad params | Check operator configuration |

---

## Diagnostic Tools

### Check file exists

```json
{"command": "ls -la ./plans/", "timeout": 10}
```

### Validate JSONL

```json
{"command": "head -5 ./input.jsonl | python -m json.tool", "timeout": 10}
```

### Check environment

```json
{"command": "echo $DASHSCOPE_API_KEY | head -c 10", "timeout": 10}
```

### View file content

```json
{"file_path": "./plans/plan_xxx.yaml"}
```

---

## Tool Execution Patterns

### Safe execution with validation

```
// 1. Validate plan first
plan_validate(plan_payload=...)

// 2. Dry run
apply_recipe(plan_path="...", dry_run=true)

// 3. Execute
apply_recipe(plan_path="...", confirm=true)
```

### Incremental spec building

```
// Build and validate each spec
build_dataset_spec(...) -> validate_dataset_spec(...)
build_process_spec(...) -> validate_process_spec(...)
build_system_spec(...) -> validate_system_spec(...)

// Then assemble
assemble_plan(...)
```

---

## File Operations for Debugging

### View log files

```json
{"file_path": "./data/log/export_xxx.txt"}
```

### View generated recipe

```json
{"file_path": "./.djx/recipes/plan_xxx.yaml"}
```

### View plan file

```json
{"file_path": "./plans/plan_xxx.yaml"}
```

---

## Environment Variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `DASHSCOPE_API_KEY` | API key | - |
| `DJA_SESSION_MODEL` | Session model | qwen3-max |
| `DJA_PLANNER_MODEL` | Planner model | qwen3-max |
| `DJA_MODEL_FALLBACKS` | Fallback models | - |
| `DJA_LLM_THINKING` | Enable thinking | true |

---

## Still Stuck?

1. Check all patches in `patches/` directory
2. Use `view_text_file` to inspect config files
3. Use `execute_shell_command` to run diagnostics
4. Try with simpler inputs to isolate the issue
