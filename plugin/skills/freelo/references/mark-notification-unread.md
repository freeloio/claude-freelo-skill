# Mark a notification as unread

Load when the user asks to mark a notification as unread.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/notification/{notification_id}/mark-as-unread"
```
