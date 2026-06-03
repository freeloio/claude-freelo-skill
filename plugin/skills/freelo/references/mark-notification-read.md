# Mark a notification as read

Load when the user asks to mark a notification as read.

```bash
# Mark read / unread
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/notification/{notification_id}/mark-as-read"
```
