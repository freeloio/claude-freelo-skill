# Rename a custom field

Load when the user asks to rename an existing custom field.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Estimate"}' \
  "$FREELO_BASE_URL/custom-field/rename/{field_uuid}"
```
