# Delete a task

Load when the user asks to delete a task. Irreversible — confirm the task name back to the user first. Prefer the finish endpoint instead unless the user explicitly says delete.

### Delete task (irreversible)

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/task/{task_id}"
```

Returns `{"result": "success"}`. **Prefer `finish` over `delete`** unless the user explicitly says "delete" — finish preserves history.
