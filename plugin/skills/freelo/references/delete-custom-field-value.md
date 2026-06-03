# Delete a custom field value on a task

Load when the user asks to clear a custom field value on a specific task.

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/custom-field/delete-value/{value_uuid}"
```
