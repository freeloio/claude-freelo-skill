# Nested subtasks are not supported

Load when the user asks for sub-subtasks or three-level nesting. The API accepts the call but returns task_id: null, leaving the entity orphaned.

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

### Nested subtasks — **not supported, do not use**

Calling `POST /task/{subtask_task_id}/subtasks` to create a subtask underneath an existing subtask returns HTTP 200, **but the response has `"task_id": null`**:

```json
{"id": 16618517, "task_id": null, "name": "nested", ...}
```

Because `task_id` is null, the resulting "nested subtask" cannot be fetched, finished, activated, or deleted through `/task/{task_id}/*`. It is effectively orphaned in the API — only visible in the Freelo UI, if at all. **Do not create subtasks of subtasks via API.** If the user asks for a three-level hierarchy, either flatten it or advise them to use the Freelo web UI.
