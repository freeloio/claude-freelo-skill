# Check out-of-office status

Load when the user asks whether a user is currently out of office, or asks for their OOO status.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```

Response when inactive: `{"out_of_office": null}`.
Response when active: `{"out_of_office": {"date_from": "...", "date_to": "..."}}`.
