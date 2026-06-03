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
# By UUID (preferred — uses an existing task label, no duplicates)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"uuid":"abc-uuid-here"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"

# By name (creates a NEW task label in the global pool with default grey color
# — see "Resolving a label UUID" below before using this on bulk operations)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"name":"Bug"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"
```

### Resolving a label UUID — read this before bulk operations

**There is no `GET /task-labels` endpoint** (returns 404). The global task-label pool is not directly listable. `/project-labels/find-available` only returns *project* labels, not task labels (the two systems are decoupled).

To get the UUID of an existing task label by name (e.g. "Urgent"), inspect a task that already carries it:

```bash
# Find any recent task that uses the label, then read its labels[]
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/task/{some_task_with_label}" \
  | jq -r '.labels[] | select(.name == "Urgent") | .uuid'
```

If no task has the label yet, or the user can't point at one, you have two paths:

1. **Create it once explicitly** with the desired name + color via [create-task-label.md](create-task-label.md), then immediately add it to any single task and capture the UUID from that task's `labels[]` in the response.
2. **Accept the by-name shortcut** — but **warn the user explicitly** that this creates a new grey-colored label in the global pool on every call, leading to label-pool clutter. Acceptable for a single ad-hoc add; avoid for any loop.

### Bulk operations — never loop with `{"name": ...}`

For "add label Urgent to all tasks in tasklist X", resolve the UUID **once** (per the section above), then loop using `{"uuid": "..."}`. Looping with `{"name": "Urgent"}` produces a fresh grey-colored duplicate on every iteration.
