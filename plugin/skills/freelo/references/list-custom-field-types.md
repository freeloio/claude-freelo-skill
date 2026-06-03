# List custom field types

Load when the user asks what custom field types are supported (number, text, date, date_time, bool, link, enum). Required as a prerequisite for create-custom-field — the type UUID comes from this call.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/custom-field/get-types"
```

Response shape: `{"custom_field_types": [{"uuid": "...", "name": "..."}]}`.

**Seven supported types** (names as returned by API):

- `number`
- `text`
- `date`
- `date_time`
- `bool`
- `link`
- `enum`
