# Create a tasklist from a template

Load when the user asks to create a tasklist from an existing tasklist template.

```bash
# Tasklist from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/tasklist/create-from-template/{template_id}"
```
