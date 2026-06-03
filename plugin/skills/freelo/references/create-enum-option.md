# Create an enum option for a custom field

Load when the user asks to add an option to an enum-type custom field. Color must be from the 26-color whitelist (see curl-conventions.md).

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Option A","color":"#e9483a"}' \
  "$FREELO_BASE_URL/custom-field-enum/create/{field_uuid}"
```
