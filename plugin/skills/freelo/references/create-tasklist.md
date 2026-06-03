# Create a tasklist

Load when the user asks to create a tasklist in a project.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"name":"Sprint 1"}' \
  "$FREELO_BASE_URL/project/{project_id}/tasklists"
```

Optional: `"budget": "100000"` (currency format, ×100).

> **Budget is write-only over the API.** The create response echoes `budget: {amount, currency}` once, but subsequent `GET /tasklist/{id}` and `GET /project/{id}` → tasklists[] responses **do not expose the budget field**. If the user needs to see a tasklist's current budget, direct them to the Freelo UI, or capture the value at create time.
