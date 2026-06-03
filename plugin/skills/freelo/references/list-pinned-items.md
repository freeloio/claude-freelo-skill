# List pinned items on a project

Load when the user asks to list pinned links on a project.

```bash
# List
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/project/{project_id}/pinned-items"
```
