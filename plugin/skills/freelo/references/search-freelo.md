# Search Freelo

Load when the user asks to find a task, subtask, project, tasklist, file, or comment by keyword. Faster than listing for resolving names to IDs.

The single most useful endpoint for resolving names to IDs.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"search_query":"deployment","entity_type":"task"}' \
  "$FREELO_BASE_URL/search"
```

**Body fields:**

- `search_query` — the search text (**required, minimum 2 characters**, must not be empty or null).
- `entity_type` — one of `task`, `subtask`, `project`, `tasklist`, `file`, `comment` (optional — omit to search all).
- `projects_ids` — array of ints to scope search (optional).
- `page` — pagination (optional, default 0). **Note:** `page` in body here, NOT `?p=` in query.

**Validation errors:**

- Empty query → `400 'search_query' must not be empty or null`
- Query with a single character → `400 'search_query' must be at least 2 characters long`
- Special / HTML characters in the query (e.g. `<script>`) are accepted as **literal search text** — no sanitization or rejection. Treat the query as an opaque string.

Response shape: `{total, count, page, per_page, data: {items: [...]}}`.

Each item: `{id, uuid, name, type, project, tasklist?, task?, state, score, highlight_name, highlight_content}`. Score is a relevance ranking.

### Search result quirks

- **`state` is an integer here**, not an object. In regular task/project endpoints you see `state: {id: 1, state: "active"}`; in search items you see `state: 1`. Map numeric values to names yourself (1 = active, 5 = finished, etc.).
- **`entity_type: "subtask"` returns items with `type: "task"`** — because subtasks are internally regular tasks with a parent. If you need to distinguish, check the item's `task.id` field (it will reference the parent task for a subtask) or re-fetch via `/task/{id}` and inspect `parent_task_id`.
- **`highlight_name` / `highlight_content`** are arrays of strings with `<em>matched</em>` tags around matches — useful for displaying search hits with emphasis.
