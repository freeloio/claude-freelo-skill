# Invite users to projects

Load when the user asks to invite people to one or more projects by email.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"emails":["a@b.com","c@d.com"],"projects_ids":[123,456]}' \
  "$FREELO_BASE_URL/users/manage-workers"
```
