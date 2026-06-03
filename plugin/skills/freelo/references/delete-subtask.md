# Delete a subtask

Load when the user asks to delete a subtask. Irreversible — confirm first.

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

```bash
# Delete
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/task/{subtask_task_id}"
```

> The endpoints `/subtask/{id}/finish`, `/subtask/{id}/activate`, `DELETE /subtask/{id}`, and `GET /subtask/{id}` all return **404** — they do not exist. Do not call them.
