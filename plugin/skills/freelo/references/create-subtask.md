# Create a subtask

Load when the user asks to add a subtask under a task. Read task_id from the response — id is not routable.

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Write tests","worker":12345,"due_date":"2026-06-15","priority_enum":"m"}' \
  "$FREELO_BASE_URL/task/{parent_task_id}/subtasks"
```

Same body rules as tasks (worker = integer, priority_enum). **Read `task_id` from the response** for subsequent operations.
