# Get task detail

Load when the user asks for full details of a specific task: labels, author, worker, state, dates, comments, cost, minutes, custom fields, estimates.

### Task detail

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/task/{task_id}"
```

Full detail — labels, author, worker, state, dates, cost, minutes, comments array, project, tasklist, estimates, custom fields.

> **`cost` and `minutes` are rolled-up**: calculated from the task's work reports (`minutes = sum of work-report minutes`, `cost = minutes × worker hourly rate`). You cannot set them directly on a task create/edit. A fresh task with no work reports has `cost: {amount: "0", currency: "CZK"}`, `minutes: 0`.
