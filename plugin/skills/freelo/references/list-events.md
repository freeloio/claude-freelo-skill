# List events (audit log)

Load when the user asks about recent activity, an audit log, or what changed on a project or by a user.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/events?projects_ids[]={project_id}&users_ids[]={user_id}&p=0"
```

Paginated with `data.events`.

**Supported query filters:**

- `projects_ids[]=N` — scope to one or more projects
- `users_ids[]=N` — events produced by specific users
- `date_from=YYYY-MM-DD` / `date_to=YYYY-MM-DD` — date range for the activity log
- `p=N` — page (0-indexed)

```bash
# Last month's activity across the whole account
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/events?date_from=2026-03-01&date_to=2026-04-01"
```

Each event is rich — includes `author` (with out-of-office info), nested `project`, `task`, `tasklist`, `comment`, `file`, `document` references (any irrelevant ones are `null`), and a `type` string like `new_project`, `new_task`, `finish_task`, etc.
