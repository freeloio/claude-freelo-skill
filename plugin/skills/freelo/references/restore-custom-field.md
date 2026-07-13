# Restore a deleted custom field

Load when the user asks to restore a previously deleted custom field.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/custom-field/restore/{field_uuid}"
```
