# Delete an enum option (safe)

Load when the user asks to delete an enum option. Safe variant — fails if the option is in use on any task. Use force-delete-enum-option.md if you need to remove it anyway.

```bash
# Delete option (safe, fails if used on tasks)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/custom-field-enum/delete/{enum_uuid}"
```
