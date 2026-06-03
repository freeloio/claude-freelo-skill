# Remove workers from a project

Load when the user asks to remove workers from a project by email.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"users_emails":["a@b.com","c@d.com"]}' \
  "$FREELO_BASE_URL/project/{project_id}/remove-workers/by-emails"
```

> **Body key is `users_emails`, not `emails`.** Sending `{"emails":[...]}` returns `400 Missing item 'users_emails'`. Note the asymmetry with the **invite** call (which uses `emails`) — they take different keys despite the parallel purpose.
