# Finish a subtask

Load when the user asks to finish or complete a subtask.

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

```bash
# Finish
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/task/{subtask_task_id}/finish"
```
