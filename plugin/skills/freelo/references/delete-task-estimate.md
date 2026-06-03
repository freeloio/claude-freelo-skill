# Delete the total time estimate on a task

Load when the user asks to remove an estimate from a task.

```bash
# Delete total estimate
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/task/{task_id}/total-time-estimate"
```

**Per-user estimates**: the public path is not yet confirmed (`/task/{id}/user-time-estimate` returns 404). If the user needs per-user estimates, recommend the Freelo UI or the OpenAPI docs until this path is validated.

The detail of an existing estimate appears in `GET /task/{id}` under `total_time_estimate` and `users_time_estimates[]`.
