# Create a project from a template

Load when the user asks to spin up a new project from an existing template.

```bash
# Project from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New Project"}' \
  "$FREELO_BASE_URL/project/create-from-template/{template_id}"
```
