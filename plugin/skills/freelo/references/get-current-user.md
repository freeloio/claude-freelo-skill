# Get current user

Load when the user asks who they are, wants to verify credentials, or needs their user ID.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/users/me"
```

Response: `{"result": "success", "user": {"id": 12345}}` — the minimal auth echo.
