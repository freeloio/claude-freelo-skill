# Get project detail

Load when the user asks for details of a specific project, including budget and time rollups.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/project/{project_id}"
```

Returns the full project with its `tasklists` and nested task IDs/names. Works for projects you own **or** are invited to.

**Top-level financial fields** (useful for reporting dashboards):

| Field | Meaning |
|---|---|
| `budget` | Configured cost budget — `{amount, currency}` |
| `minutes_budget` | Configured time budget in minutes (int) |
| `real_minutes_spent` | Actual minutes logged via work reports across the project (rolled up) |
| `real_cost` | Actual cost = logged minutes × hourly rates — `{amount, currency}` |

These are **read-only rollups**. They update automatically as work reports accumulate.
