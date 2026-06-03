# Create a task label (global pool)

Load when the user asks to create a new task label in the global pool. Body MUST be wrapped in a labels array.

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### Task labels — global pool

```bash
# Create one or more task-labels (they go into the global pool).
# Body MUST be wrapped in a "labels" array — sending {"name":..., "color":...} at the top level returns 400.
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"name":"Bug","color":"#e9483a"}]}' \
  "$FREELO_BASE_URL/task-labels"
```

Response: `{"result": "success"}`. The created labels do not come back in the body — re-fetch via `/task-labels/add-to-task/{tid}` response (which includes the UUID) or by adding the label to a task and reading the task's `labels[]`.
