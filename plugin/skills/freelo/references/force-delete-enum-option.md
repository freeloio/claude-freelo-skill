# Force-delete an enum option (removes from all tasks)

Load when the user asks to remove an enum option AND clear it from all tasks currently using it. Destructive — confirm with the user first.

```bash
# Force-delete (removes from all tasks too)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/custom-field-enum/force-delete/{enum_uuid}"
```
