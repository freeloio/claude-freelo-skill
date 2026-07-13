# List template projects

Load when the user asks to list their project templates.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/template-projects"
```

Paginated shape: `{data: {template_projects: [...]}, ...}`.
