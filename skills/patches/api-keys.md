---
name: api-keys
description: API key configuration issues
when:
  - error contains ["API", "key", "unauthorized", "401", "authentication", "credential"]
---

# Patch: API Key Configuration

## Problem

API or authentication errors when executing tools that require LLM access.

## Symptoms

- "API key not found"
- "Unauthorized" or "401"
- "Authentication failed"

---

## Solution

### Check current configuration

```json
{"command": "echo $DASHSCOPE_API_KEY | head -c 10", "timeout": 10}
```

### Set API key via shell

```json
{"command": "export DASHSCOPE_API_KEY='your_key'", "timeout": 10}
```

---

## API Key Priority

The system checks in this order:
1. `DASHSCOPE_API_KEY`
2. `MODELSCOPE_API_TOKEN`

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `DASHSCOPE_API_KEY` | Primary API key |
| `MODELSCOPE_API_TOKEN` | Alternative API key |
| `DJA_OPENAI_BASE_URL` | Custom endpoint |
| `DJA_SESSION_MODEL` | Session model |
| `DJA_PLANNER_MODEL` | Planner model |
| `DJA_MODEL_FALLBACKS` | Fallback models |
| `DJA_LLM_THINKING` | Enable thinking mode |

---

## Thinking Mode

Some models don't support the thinking flag:

```json
{"command": "export DJA_LLM_THINKING=false", "timeout": 10}
```

---

## Verify Configuration

Test with a simple retrieve:

```json
{
  "intent": "test",
  "top_k": 1,
  "mode": "vector"
}
```

Vector mode doesn't require LLM, so if this works but LLM mode fails, the API key is the issue.
