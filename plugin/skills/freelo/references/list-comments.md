# List comments

Load when the user asks to list comments across the account, on a project, or on a task. Includes the richer comment shape when read via /task/{id}.

### List all comments (paginated)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/all-comments"
```

Filters: `?projects_ids[]=N`, `?tasks_ids[]=N`, `?p=N`. Shape: `{data: {comments: [...]}, ...}`.

### Comment shape inside `GET /task/{id}` (richer)

When you read a task's `comments[]` via task detail, each comment carries more fields than a direct create response:

```json
{
  "id": 31987920,
  "content": "<div>...</div>",
  "date_add": "2026-04-14T09:48:07+02:00",
  "author": { "id": 12345, "fullname": "John Doe" },
  "is_description": true,                 // true ONLY for the task's seed comment (from task create with `comment:{...}`), which doubles as the description
  "comments_reactions": [],               // emoji reactions added in the Freelo UI
  "files": [ {...file metadata rich shape...} ]
}
```

- `is_description: true` marks the initial comment that was submitted via `comment:{...}` in the task create body — this is the same thing that appears under `/task/{id}/description`. Regular comments added later have `is_description: false` (or the field is absent).
- `comments_reactions` is an array of user reactions (emoji) set through the UI — normally empty over API.
