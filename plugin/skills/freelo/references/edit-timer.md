# Edit the active timer session

Load when the user asks to switch the timer to a different task, or change the note on the active session.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":67890,"note":"Switched focus"}' \
  "$FREELO_BASE_URL/timetracking/edit"
```
