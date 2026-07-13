# List issued invoices

Load when the user asks to list invoices, optionally scoped to a project.

```bash
# List issued invoices
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/issued-invoices?projects_ids[]={project_id}"
```
