# List work reports

Load when the user asks to list work reports filtered by project, user, or pagination. Known caveat: listing may return total: 0 in limited API contexts even when reports exist.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/work-reports?projects_ids[]={project_id}&users_ids[]={user_id}&p=0"
```

Response shape: `{total, count, page, per_page, data: {reports: [...]}}`.

> **Caveat:** in limited-scope API contexts the listing may return `total: 0` even when reports exist. If this happens, fetch individual reports via `/work-reports/{report_id}` using IDs you have from create/timer-stop responses, or inspect a task's detail for its rolled-up `minutes`.
