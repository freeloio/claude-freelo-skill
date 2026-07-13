# List projects

Load when the user asks to list their projects, all projects, archived projects, template projects, or cannot find a project.

Freelo has **three project-listing endpoints** with different shapes and scopes — know when to use which:

### 1. `/projects` — projects where you are the **owner** (bare array)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/projects"
```

Bare array. Each element includes `tasklists` with nested task previews. **Does NOT include projects where you are only invited as a worker.**

### 2. `/all-projects` — same owner scope, but paginated

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/all-projects?p=0"
```

Response shape: `{total, count, page, per_page, data: {projects: [...]}}`.

### 3. `/invited-projects` — projects where you are a worker (invited)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/invited-projects?p=0"
```

Response shape: `{total, count, page, per_page, data: {invited_projects: [...]}}`.

> **When the user mentions a project and you cannot find it via `/projects`, also check `/invited-projects`.**

### Project detail

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/project/{project_id}"
```

Returns the full project with its `tasklists` and nested task IDs/names. Works for projects you own **or** are invited to.

**Top-level financial fields** (useful for reporting dashboards):

| Field | Meaning |
|---|---|
| `budget` | Configured cost budget — `{amount, currency}` |
| `minutes_budget` | Configured time budget in minutes (int) |
| `real_minutes_spent` | Actual minutes logged via work reports across the project (rolled up) |
| `real_cost` | Actual cost = logged minutes × hourly rates — `{amount, currency}` |

These are **read-only rollups**. They update automatically as work reports accumulate.

### Archived projects

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/all-projects?state=archived"
```

### Template projects

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/template-projects"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {template_projects: [...]}}`.
