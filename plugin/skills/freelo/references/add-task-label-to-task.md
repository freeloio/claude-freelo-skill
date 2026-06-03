# Add a task label to a task

Load when the user asks to apply a task label to a specific task — by UUID (uses existing label) or by name (creates new in the global pool).

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### Add task label(s) to a task

```bash
# By UUID (preferred — uses an existing task label)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"uuid":"abc-uuid-here"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"

# By name (creates a NEW task label in the global pool with default grey color)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"name":"Bug"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"
```
