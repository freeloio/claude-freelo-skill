# Stop the timer

Load when the user asks to stop their timer. If a task was attached, a work report is auto-created on stop.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/timetracking/stop"
```

Returns the created work report (when a task was attached): `{id, minutes, note, task, author, worker, cost, date_reported, date_add}`.
