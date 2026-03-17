---
name: retrieve
description: Search operators using retrieve_operators tool
when:
  - user_intent contains ["search operator", "what operators", "find operator", "搜索算子", "有哪些算子", "查找"]
---

# Retrieve Skill

Search Data-Juicer operators using the `retrieve_operators` tool.

## Tool

**Tool:** `retrieve_operators`

```json
{
  "intent": "filter text by length",
  "top_k": 10,
  "mode": "auto",
  "dataset_path": ""
}
```

---

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `intent` | string | Required | Natural language search query |
| `top_k` | integer | 10 | Maximum candidates to return (min: 1) |
| `mode` | string | "auto" | Retrieval mode: `auto`, `llm`, or `vector` |
| `dataset_path` | string | "" | Optional dataset for context |

---

## Examples

### Basic search

```json
{
  "intent": "filter text by length",
  "top_k": 10
}
```

### Search with dataset context

```json
{
  "intent": "deduplicate multimodal data",
  "top_k": 15,
  "dataset_path": "./data/multimodal.jsonl"
}
```

### LLM-based retrieval

```json
{
  "intent": "remove low quality text",
  "mode": "llm",
  "top_k": 8
}
```

---

## Return Value

```json
{
  "ok": true,
  "candidates": [
    {
      "operator_name": "text_length_filter",
      "description": "Filter samples by text length",
      "score": 0.95
    },
    {
      "operator_name": "document_deduplicator",
      "description": "Remove duplicate documents",
      "score": 0.87
    }
  ],
  "source": "vector",
  "notes": "..."
}
```

---

## Retrieval Modes

| Mode | Description |
|------|-------------|
| `auto` | Automatically choose best method |
| `vector` | Semantic vector search (fast) |
| `llm` | LLM-based matching (more accurate) |

---

## Use Cases

### 1. Exploration before planning

```json
// Find available operators
{"intent": "text quality filtering", "top_k": 15}

// Then use results in build_process_spec
```

### 2. Context for custom operator development

```json
// Find similar operators
{"intent": "sentiment analysis", "top_k": 5}

// Then pass results to develop_operator
```

### 3. Check operator availability

```json
{"intent": "specific_operator_name", "top_k": 1}
```

---

## Common Search Intents

| Goal | Intent Example |
|------|----------------|
| Text length | "filter text by length" |
| Deduplication | "remove duplicate documents" |
| Language | "filter by language" |
| Quality | "filter low quality text" |
| Cleaning | "normalize whitespace" |
| Multimodal | "deduplicate images" |

---

## Integration with Planning

Use retrieved operators in `build_process_spec`:

```json
// After retrieve_operators returns candidates
{
  "operators": [
    {"name": "text_length_filter", "params": {"min_len": 50}},
    {"name": "document_deduplicator", "params": {}}
  ]
}
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| No results | Try broader/different intent |
| Wrong operators | Be more specific in intent |
| Slow retrieval | Use `mode: "vector"` for speed |

See [debug.md](debug.md) for detailed troubleshooting.
