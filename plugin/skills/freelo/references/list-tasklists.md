# List tasklists

Load when the user asks to list tasklists in a project or across the account.

### List all tasklists (paginated)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/all-tasklists"
```

Response: `{total, count, page, per_page, data: {tasklists: [...]}}`.

### List tasklists in a project

Use `GET /project/{id}` and read `.tasklists` from the response.

### Tasklist detail (includes tasks)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/tasklist/{tasklist_id}"
```
