# Set a reminder on a task

Load when the user asks to set a reminder on a task.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"remind_at":"2026-06-15T09:00:00+02:00"}' \
  "$FREELO_BASE_URL/task/{task_id}/reminder"
```
