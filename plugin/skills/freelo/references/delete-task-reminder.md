# Delete a reminder on a task

Load when the user asks to cancel a reminder on a task.

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/task/{task_id}/reminder"
```
