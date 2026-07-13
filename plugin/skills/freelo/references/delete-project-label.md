# Delete a project label

Load when the user asks to remove a project label. Cleanest removal — prefer this over remove-from-project.

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### Delete a project label (cleanest removal)

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/project-labels/{label_id}"
```

Returns `{"result": "success"}`. **Prefer this over `/project-labels/remove-from-project/...`** — that endpoint currently requires extra fields and is unreliable.
