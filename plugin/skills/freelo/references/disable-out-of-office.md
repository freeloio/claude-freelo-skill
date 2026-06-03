# Disable out-of-office

Load when the user asks to cancel their OOO.

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```
