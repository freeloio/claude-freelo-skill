# Edit a task

Load when the user asks to rename a task, change its priority, reassign worker, or change dates.

### Edit task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"name":"New name","priority_enum":"m","worker":12345,"due_date":"2026-07-01","due_date_end":"2026-07-05"}' \
  "$FREELO_BASE_URL/task/{task_id}"
```

Send only fields you want to change.

> **Due-date range quirk:** When changing the end date, **always send both `due_date` AND `due_date_end` together**. Sending only `due_date_end` currently causes the API to overwrite `due_date` with the value and null out `due_date_end`. Always include the full range in edits.
