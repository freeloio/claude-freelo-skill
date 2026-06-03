# List enum options for a custom field

Load when the user asks what enum options exist for an enum-type custom field.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/custom-field-enum/get-for-custom-field/{field_uuid}"
```
