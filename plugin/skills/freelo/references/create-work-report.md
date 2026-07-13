# Create a work report manually

Load when the user asks to log time against a task manually (without using the timer).

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"minutes":120,"date_reported":"2026-06-15","note":"Implementation","worker_id":12345}' \
  "$FREELO_BASE_URL/task/{task_id}/work-reports"
```

- `minutes` — required.
- `date_reported` — required, `YYYY-MM-DD`.
- `note`, `worker_id` — optional. If `worker_id` is omitted, the report is attributed to the current user.
