# Start a timer

Load when the user asks to start tracking time on a task or in general. Only one timer per user — starting a second returns 409.

> Only **one** timer runs per user at a time. Starting a new one while another is active returns HTTP 409.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"note":"Implementing feature X"}' \
  "$FREELO_BASE_URL/timetracking/start"
```

Both fields optional — you can track "free" time without a task. Response is minimal: `{"uuid": "..."}`.
