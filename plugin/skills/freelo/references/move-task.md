# Move a task to another tasklist

Load when the user asks to move a task between tasklists.

### Move task to another tasklist

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/task/{task_id}/move/{target_tasklist_id}"
```
