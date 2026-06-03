# List tasks

Load when the user asks for a list of tasks: all tasks, today's tasks, overdue, finished, by project, by tasklist, by worker, by label, or by text query.

### List all tasks (paginated — the workhorse)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/all-tasks"
```

Shape: `{total, count, page, per_page, data: {tasks: [...]}}`. **Most powerful filtering endpoint** — query params:

- `?projects_ids[]=1&projects_ids[]=2` — limit to projects
- `?tasklists_ids[]=N` — limit to tasklists
- `?workers_ids[]=N` — tasks for a specific user
- `?states[]=active&states[]=finished` — filter by state
- `?q=search+term` — text search
- `?due_date_from=2026-01-01&due_date_to=2026-12-31` — due-date range
- `?labels_uuids[]=<uuid>` — filter by task label
- `?p=N` — pagination (0-indexed)

### Finished / overdue tasks — use filters on `/all-tasks`

There is no separate endpoint; filter `/all-tasks` with `states[]` or a date range:

```bash
# Finished tasks in a project
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/all-tasks?states[]=finished&projects_ids[]={project_id}"

# Overdue — active tasks with due_date before today
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/all-tasks?states[]=active&due_date_to=2026-04-14"
```

> `/all-tasks/finished` and `/all-tasks/overdue` as separate paths return **404** — do not use them.

### Tasks in a tasklist

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/project/{project_id}/tasklist/{tasklist_id}/tasks"
```

Response shape: **bare array**, not paginated.
