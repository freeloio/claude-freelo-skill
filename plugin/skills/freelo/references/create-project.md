# Create a project

Load when the user asks to create a new project.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Project Name","currency_iso":"CZK"}' \
  "$FREELO_BASE_URL/projects"
```

`currency_iso` optional (default `CZK`). Values: `CZK`, `EUR`, `USD`.
