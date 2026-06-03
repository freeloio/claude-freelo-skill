# Create a task from a template

Load when the user asks to create a task from an existing task template.

```bash
# Task from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/task/create-from-template/{template_id}"
```
