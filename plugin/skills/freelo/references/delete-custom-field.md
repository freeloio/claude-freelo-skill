# Delete a custom field

Load when the user asks to delete a custom field. The field can be restored via the restore endpoint.

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/custom-field/delete/{field_uuid}"
```
