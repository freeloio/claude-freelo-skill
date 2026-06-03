# List work reports

Load when the user asks to list work reports filtered by project, user, date range, or pagination — including "how many hours this week", weekly / monthly team reports, or workload summaries.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/work-reports?projects_ids[]={project_id}&users_ids[]={user_id}&p=0"
```

Response shape: `{total, count, page, per_page, data: {reports: [...]}}`.

Each report item carries: `id`, `minutes`, `note`, `date_reported`, `worker: {id, fullname}`, `project: {id, name}`, `task: {id, name}`, `cost: {amount, currency}`.

## Filters

| Query param | Purpose | Example |
|---|---|---|
| `projects_ids[]=N` | Limit to project(s) — repeat for multiple | `?projects_ids[]=580898` |
| `users_ids[]=N` | Limit to worker(s) — repeat for multiple | `?users_ids[]=267807` |
| `tasks_ids[]=N` | Limit to specific tasks | `?tasks_ids[]=29234425` |
| `tasks_labels[]=<uuid>` | Filter by task label UUID | `?tasks_labels[]=abc-def-…` |
| `date_reported_range[date_from]=YYYY-MM-DD` | Reports on/after this date | see below |
| `date_reported_range[date_to]=YYYY-MM-DD` | Reports on/before this date | see below |
| `p=N` | Pagination, 0-indexed | `?p=1` |

> **Important — date filter syntax.** The server expects the bracket form `date_reported_range[date_from]` and `date_reported_range[date_to]`. The flat form `date_reported_from` / `date_reported_to` is **silently ignored** (verified live against `api.freelo.io` 2026-05-15). In a URL you'll URL-encode the brackets as `%5B` and `%5D`.

## Weekly / monthly team report — the canonical pattern

User asks "Kolik hodin tento týden odpracoval tým na projektu X?" or similar:

```bash
# Week of 2026-06-01 to 2026-06-07 on project 580898
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/work-reports?projects_ids[]=580898&date_reported_range%5Bdate_from%5D=2026-06-01&date_reported_range%5Bdate_to%5D=2026-06-07"
```

Sum the `minutes` field per `worker.id` locally. **Use `minutes`, not `time_spent`** (the latter does not exist on the response). Group and convert to hours: `minutes / 60`.

## Caveats

- **Default range.** Without `date_reported_range[*]`, the server applies a narrow default range and freshly-created reports may not appear. Always pass an explicit range for "this week", "this month", etc.
- **Limited-scope `total: 0` quirk.** In some API contexts the listing returns `total: 0` even when reports exist. If this happens, fetch individual reports via `/work-reports/{report_id}` using IDs you have from create / timer-stop responses, or inspect the task detail for its rolled-up `minutes`.
- **No `worker` URL pattern.** Workers are not entities with a public Freelo URL — render their names as plain bold text in the response, not as a link.
