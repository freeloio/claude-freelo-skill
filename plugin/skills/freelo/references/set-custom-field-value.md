# Set a custom field value on a task

Load when the user asks to set a value for a text, number, date, date_time, bool, or link field on a task. For enum fields, use set-enum-value-on-task.md instead.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"custom_field_uuid":"abc-uuid","value":"42"}' \
  "$FREELO_BASE_URL/custom-field/add-or-edit-value"
```
