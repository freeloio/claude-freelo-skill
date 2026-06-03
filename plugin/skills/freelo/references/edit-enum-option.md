# Edit an enum option

Load when the user asks to rename or recolor an enum option.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"name":"Option B","color":"#62d26f"}' \
  "$FREELO_BASE_URL/custom-field-enum/change/{enum_uuid}"
```
