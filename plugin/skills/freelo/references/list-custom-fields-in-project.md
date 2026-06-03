# List custom fields in a project

Load when the user asks to list the custom fields defined on a project. Response includes is_commander flag indicating edit/delete permission.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/custom-field/find-by-project/{project_id}"
```

Response shape: `{"custom_fields": [...], "is_commander": bool}`. The `is_commander` flag tells you whether the current user has permission to edit/delete custom fields in this project.
