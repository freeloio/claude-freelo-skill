# Delete a project

Load when the user asks to delete a project. Irreversible — confirm the project name back to the user before calling this.

```bash
# Delete (irreversible!)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/project/{project_id}"
```
