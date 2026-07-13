# Add a label to a project

Load when the user asks to create a new label on a project. All three fields (name, color from whitelist, is_private) are required.

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### Add a label to a project

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Priority","color":"#e9483a","is_private":false}' \
  "$FREELO_BASE_URL/project-labels/add-to-project/{project_id}"
```

All three fields are required: `name`, `color` (from whitelist), `is_private` (bool). Response: `{"result": "success"}`.
