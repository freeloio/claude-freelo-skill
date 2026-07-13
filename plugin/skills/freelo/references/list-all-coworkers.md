# List all coworkers

Load when the user asks to list all users on the account.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/users"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {users: [...]}}`.
