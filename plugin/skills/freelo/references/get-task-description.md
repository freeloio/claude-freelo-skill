# Get the description of a task

Load when the user asks to read the current description of a task.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/task/{task_id}/description"
```

Response includes {id, content, date_add, files} — the description has its own id and can have file attachments, just like a comment.
