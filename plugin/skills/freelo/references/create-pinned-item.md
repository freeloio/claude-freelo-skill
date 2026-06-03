# Pin a link to a project

Load when the user asks to pin an external link to a project.

```bash
# Create
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"link":"https://example.com/doc","title":"Design spec"}' \
  "$FREELO_BASE_URL/project/{project_id}/pinned-items"
```
