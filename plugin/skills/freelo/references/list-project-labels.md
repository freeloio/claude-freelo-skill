# List available project labels

Load when the user asks to list labels defined on a project.

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### List available project labels

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/project-labels/find-available"
```

Response shape: `{"labels": [...]}` — a dict with a `labels` key, not a bare array. Each label has `{id (int), name, color, is_private, users_id, usage_count, can_be_public, can_be_edited}`.
