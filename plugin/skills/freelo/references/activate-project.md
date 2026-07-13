# Reactivate a project

Load when the user asks to reactivate or unarchive a project.

```bash
# Reactivate
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/project/{project_id}/activate"
```
