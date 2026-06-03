# List subtasks of a task

Load when the user asks to list subtasks under a task.

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/task/{task_id}/subtasks"
```

**Response shape — paginated**: `{total, count, page, per_page, data: {subtasks: [...]}}`.

Each subtask item has both `id` (subtask-specific) and `task_id` (usable task ID).
