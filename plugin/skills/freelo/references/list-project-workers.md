# List workers on a project

Load when the user asks who is on a specific project.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/project/{project_id}/workers"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {workers: [...]}}`.
