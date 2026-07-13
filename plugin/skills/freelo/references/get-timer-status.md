# Get current timer status

Load when the user asks what they are tracking, or whether a timer is running.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/timetracking/status"
```

Returns the full active session: `{uuid, date_reported, task: {id, name, project, tasklist}, note, cost, is_billable, project_setting, ...}`.
