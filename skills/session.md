---
name: session
description: Conversational interface using dj-agents (orchestrates atomic tools)
when:
  - user prefers conversational/interactive mode
  - user_intent contains ["interactive", "conversational", "chat", "对话", "交互", "dj-agents"]
---

# Session Skill

Start conversational interface (`dj-agents`) for interactive Data-Juicer operations.

**Note:** The session agent internally orchestrates the same atomic tools described in other skills.

## Start Session

Use `execute_shell_command` to start the session:

```json
{
  "command": "dj-agents --dataset ./input.jsonl --export ./output.jsonl",
  "timeout": 3600
}
```

---

## Command Options

| Option | Default | Description |
|--------|---------|-------------|
| `--dataset` | Required | Input dataset path |
| `--export` | Required | Output dataset path |
| `--ui` | tui | Interface: `tui` or `plain` |
| `--verbose` | false | Verbose output |

---

## Examples

### Default TUI mode

```json
{
  "command": "dj-agents --dataset ./data/input.jsonl --export ./data/output.jsonl"
}
```

### Plain terminal mode

```json
{
  "command": "dj-agents --ui plain --dataset ./data/input.jsonl --export ./data/output.jsonl"
}
```

---

## Session Controls

| Key | Action |
|-----|--------|
| `Ctrl+C` | Interrupt current turn (can continue) |
| `Ctrl+D` | Exit session |

---

## Internal Tool Chain

The session agent uses the same atomic tools:

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

---

## When to Use Session vs Direct Tools

| Scenario | Recommendation |
|----------|----------------|
| Simple, well-defined task | Use atomic tools directly |
| Complex, exploratory task | Use session |
| Need iterative refinement | Use session |
| Automation/scripting | Use atomic tools |
| Learning the system | Use session |

---

## Environment Configuration

```json
// Use execute_shell_command to set env vars if needed
{
  "command": "export DASHSCOPE_API_KEY=xxx && dj-agents --dataset ./input.jsonl --export ./output.jsonl"
}
```

### Required Variables

| Variable | Purpose |
|----------|---------|
| `DASHSCOPE_API_KEY` | API credential |

### Optional Variables

| Variable | Purpose |
|----------|---------|
| `DJA_SESSION_MODEL` | Model for session |
| `DJA_MODEL_FALLBACKS` | Fallback models |

---

## Alternative: Direct Tool Orchestration

Instead of using `dj-agents`, you can directly orchestrate the atomic tools:

```
// Step 1: Inspect
inspect_dataset(dataset_path="./input.jsonl")

// Step 2: Retrieve
retrieve_operators(intent="clean text")

// Step 3-8: Build specs and plan
build_dataset_spec(...)
build_process_spec(...)
build_system_spec(...)
assemble_plan(...)
plan_validate(...)
plan_save(...)

// Step 9: Execute
apply_recipe(plan_path="./plans/plan.yaml")
```

This gives you more control compared to the conversational interface.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Session won't start | Check API key is set |
| Model errors | Set `DJA_SESSION_MODEL` |
| Session interrupted | `Ctrl+C` to resume, `Ctrl+D` to exit |

See [debug.md](debug.md) for detailed troubleshooting.
