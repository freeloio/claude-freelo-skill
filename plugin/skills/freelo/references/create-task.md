# Create a task

Load when the user asks to create a new task in a tasklist.

### Create task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{
    "name":"Fix login bug",
    "worker":12345,
    "due_date":"2026-06-15",
    "due_date_end":"2026-06-20",
    "priority_enum":"h",
    "comment":{"content":"Initial note"}
  }' \
  "$FREELO_BASE_URL/project/{project_id}/tasklist/{tasklist_id}/tasks"
```

**Body rules:**

- `worker` is an **integer** user ID. Passing `{"id": N}` returns HTTP 400.
- `priority_enum` accepts `"l"` (low), `"m"` (medium / default), or `"h"` (high). The field `priority` (without `_enum`) is silently ignored on create.
- `due_date` and `due_date_end` accept `YYYY-MM-DD` or ISO 8601 with `+02:00` / `Z`.
- Optional `comment` seeds an initial comment.
