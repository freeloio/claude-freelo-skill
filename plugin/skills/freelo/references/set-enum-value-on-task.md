# Set an enum value on a task

Load when the user asks to set an enum-type custom field on a task to a specific option.

```bash
# Set enum value on a task
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"custom_field_uuid":"abc-uuid","custom_field_enum_uuid":"def-uuid"}' \
  "$FREELO_BASE_URL/custom-field/add-or-edit-enum-value"
```
