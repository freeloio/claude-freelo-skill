# Finish a task

Load when the user asks to finish, complete, or close a task.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/task/{task_id}/finish"
```

Prefer `finish` over `delete` unless the user explicitly says delete — finish preserves history.
