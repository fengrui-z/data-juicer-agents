---
name: data-juicer
description: Atomic tools for AI agents to process datasets with Data-Juicer operators
auto_load: true
---

# Data Juicer Skills - Router

Load specific skills based on context state. This file is always loaded as the entry point.

## Available Tools

| Category | Tools |
|----------|-------|
| Context | `inspect_dataset` |
| Plan | `build_dataset_spec`, `build_process_spec`, `build_system_spec`, `validate_*`, `assemble_plan`, `plan_validate`, `plan_save` |
| Retrieve | `retrieve_operators` |
| Apply | `apply_recipe` |
| Dev | `develop_operator` |
| Files | `view_text_file`, `write_text_file`, `insert_text_file` |
| Process | `execute_shell_command`, `execute_python_code` |

---

## Context -> Skill Mapping

| Context State | Load Skill | Action |
|---------------|------------|--------|
| User wants to process dataset + no plan exists | [plan.md](plan.md) | Build and save plan |
| Plan file exists (.yaml) | [apply.md](apply.md) | Execute the plan |
| User asks about available operators | [retrieve.md](retrieve.md) | Search operators |
| User needs custom operator | [dev.md](dev.md) | Generate operator scaffold |
| User prefers conversational mode | [session.md](session.md) | Start dj-agents |
| Tool execution failed / error occurred | [debug.md](debug.md) | Troubleshoot issues |

---

## Quick Decision Tree

```
User Request
    │
    ├─ "Process/clean/filter my dataset"
    │   └─> plan.md (multi-step) -> apply.md
    │
    ├─ "I need a custom operator"
    │   └─> dev.md -> plan.md -> apply.md
    │
    ├─ "What operators are available?"
    │   └─> retrieve.md
    │
    ├─ "I want interactive mode"
    │   └─> session.md
    │
    └─ "Something failed / error"
        └─> debug.md -> patches/
```

---

## Standard Workflows

### Workflow 1: Process Dataset (Tool Chain)

```
inspect_dataset
    -> retrieve_operators
    -> build_dataset_spec
    -> build_process_spec
    -> build_system_spec
    -> assemble_plan
    -> plan_validate
    -> plan_save
    -> apply_recipe
```

### Workflow 2: Custom Operator Development

```
retrieve_operators (find similar)
    -> develop_operator
    -> (edit generated code)
    -> plan workflow with custom_operator_paths
```

### Workflow 3: Quick Operator Search

```
retrieve_operators
```

---

## Tool Quick Reference

### inspect_dataset
```json
{"dataset_path": "./input.jsonl", "sample_size": 20}
```

### retrieve_operators
```json
{"intent": "filter by text length", "top_k": 10, "mode": "auto"}
```

### apply_recipe
```json
{"plan_path": "./plans/plan_xxx.yaml", "dry_run": false, "timeout": 300, "confirm": true}
```

### develop_operator
```json
{"intent": "filter by sentiment", "operator_name": "sentiment_filter", "output_dir": "./custom_ops", "operator_type": "filter", "smoke_check": true}
```

---

## Skill Index

| Skill | When to Load |
|-------|--------------|
| [plan.md](plan.md) | Build plan from intent (multi-step) |
| [apply.md](apply.md) | Execute existing plan |
| [retrieve.md](retrieve.md) | Explore operators |
| [dev.md](dev.md) | Create custom operators |
| [session.md](session.md) | Conversational interface |
| [debug.md](debug.md) | Troubleshoot errors |

### Patches (Load on specific errors)

| Patch | Trigger |
|-------|---------|
| [patches/api-keys.md](patches/api-keys.md) | API/auth errors |
| [patches/timeout.md](patches/timeout.md) | Timeout errors |
| [patches/custom-ops.md](patches/custom-ops.md) | Operator not found |
| [patches/dataset.md](patches/dataset.md) | Dataset format errors |
