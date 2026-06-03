# Set the total time estimate on a task

Load when the user asks to set a time estimate on a task. PAID FEATURE — returns HTTP 402 on free plans.

### Task estimates — paid feature

Task time estimates are available only on paid Freelo plans. Accounts without the feature receive **HTTP 402 Payment Required** with `{"errors": ["Payment required. Your plan has been exceeded."]}`.

```bash
# Set total estimate (body key is `minutes`)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"minutes":480}' \
  "$FREELO_BASE_URL/task/{task_id}/total-time-estimate"
```
