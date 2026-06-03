# List files in a project

Load when the user asks to list files, documents, links, or directories on a project. Response data key is items, not docs_and_files.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/all-docs-and-files?projects_ids[]={project_id}"
```

Filter by `?type=file` (or `directory`, `link`, `document`).
Response shape: `{total, count, page, per_page, data: {items: [...]}}` — note the `items` key.
