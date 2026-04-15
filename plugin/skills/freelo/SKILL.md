---
name: freelo
description: Use when interacting with the Freelo.io project management API — managing projects, tasklists, tasks, subtasks, comments, time tracking, work reports, labels, custom fields, notes, files, templates, pinned items, workers, notifications, events, invoices, or searching in Freelo. Trigger this skill whenever the user mentions Freelo, wants to manage tasks or projects in Freelo, track time, create work reports, check invoices, or do anything related to Freelo.io — even if they do not explicitly say "API".
---

# Freelo API

Freelo.io is a project and task management tool. This skill enables you to interact with Freelo's REST API.

**API reference:** <https://api.freelo.io/docs/v1/freelo-api>

---

## Authentication

Use HTTP Basic Auth with credentials from environment variables (configured in `~/.claude/settings.json`):

- **Username**: `$FREELO_EMAIL`
- **Password**: `$FREELO_API_KEY`
- **Base URL**: `$FREELO_BASE_URL` (default: `https://api.freelo.io/v1`)

Every request **must** include a `User-Agent` header — the API rejects requests without it. **Use exactly `Freelo-Claude-Skill/1.0.0`** — Freelo backend uses this specific value to identify skill-originated traffic for usage analytics. Do not substitute with a custom User-Agent; it would make the request invisible to adoption telemetry.

> **This skill is built for Claude Code** (terminal or desktop in Code mode). Credentials must come from environment variables. If `$FREELO_EMAIL` or `$FREELO_API_KEY` is unset, stop and tell the user to add them to `~/.claude/settings.json` and restart Claude Code — do not ask for credentials in the conversation.

### Base curl pattern

```bash
FREELO_BASE_URL="${FREELO_BASE_URL:-https://api.freelo.io/v1}"

curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" \
  -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  "$FREELO_BASE_URL/{endpoint}"
```

For POST/PUT requests, add `-d '{"key": "value"}'` with the JSON body.

> **Important:** Use **double quotes** around headers containing shell variables (`"User-Agent: ..."`), otherwise the variable will not expand. Use single quotes only for the JSON body (`-d '{...}'`) so you do not need to escape quotes inside JSON.

### Verify credentials

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/users/me"
```

Returns `{"result": "success", "user": {"id": ...}}` on success, `401` on failure.

---

## API Basics

| Property | Value |
|----------|-------|
| Base URL | `$FREELO_BASE_URL` (default: `https://api.freelo.io/v1`) |
| Format | JSON (UTF-8) — Czech diacritics and emoji pass through reliably |
| Auth | HTTP Basic (email:api_key) |
| Rate Limit | **25 requests/minute** — on 429, wait 60 seconds |
| Timezone | Europe/Prague (`+01:00` winter, `+02:00` summer) |

### Response shapes — four patterns

Freelo's API does **not** use a single envelope. Know the shape per endpoint:

#### 1. Paginated with named data key (most `/all-*` endpoints)

```json
{
  "total": 42,
  "count": 25,
  "page": 0,
  "per_page": 25,
  "data": { "tasks": [ ... ] }
}
```

The key inside `data` varies by endpoint:

| Endpoint | `data` key |
|---|---|
| `/all-tasks` | `tasks` |
| `/all-comments` | `comments` |
| `/all-tasklists` | `tasklists` |
| `/all-notifications` | `notifications` |
| `/all-docs-and-files` | `items` |
| `/work-reports` | `reports` |
| `/events` | `events` |
| `/issued-invoices` | `issued_invoices` |
| `/all-projects` | `projects` |
| `/invited-projects` | `invited_projects` |
| `/task/{id}/subtasks` | `subtasks` |
| `/search` | `items` |
| `/users` | `users` |
| `/project/{id}/workers` | `workers` |
| `/template-projects` | `template_projects` |

#### 2. Bare array — **no pagination, no envelope**

```json
[ {...}, {...}, {...} ]
```

Used by: `GET /projects`, `GET /project/{id}/tasklist/{tlid}/tasks`, `GET /project/{id}/pinned-items`.

#### 3. Object with a named collection key (no pagination wrapper)

```json
{ "labels": [ ... ] }
```

Used by:

- `GET /project-labels/find-available` → `{"labels": [...]}`
- `GET /custom-field/get-types` → `{"custom_field_types": [...]}`
- `GET /custom-field/find-by-project/{id}` → `{"custom_fields": [...], "is_commander": bool}`
- `GET /user/{id}/out-of-office` → `{"out_of_office": null | {...}}`

#### 4. Single entity / result

- Entity detail (`GET /project/{id}`, `GET /task/{id}`, etc.) — returns the entity object directly.
- Auth check — `{"result": "success", "user": {...}}`.
- Mutations — either `{"result": "success"}` or the updated/created entity.

### Error shapes — **two** different patterns

| HTTP | Shape | Example |
|---|---|---|
| 400, 422 | `{"errors": ["msg1", "msg2"]}` (plural, array of strings) | `{"errors": ["Missing item 'is_private'."]}` |
| 404 | `{"error": "..."}` (singular, string) | `{"error": "Page not found, HTTP code 404."}` |
| 401 | `{"error": "..."}` (singular) | Usually "Unauthorized" |

When parsing errors, check for **both** `errors` (array) and `error` (string).

### Pagination

- **Query-string endpoints** use `?p=N` (0-indexed): e.g. `GET /all-tasks?p=1&projects_ids[]=123`.
- **POST body endpoints** (only `/search` today) use `{"page": N}` in the JSON body.
- Default page size: **25** for most `/all-*` endpoints, **100** for `/search`.
- If `total > (page+1) * per_page`, there is a next page.

### Dates & times — **three formats** to recognize

| Context | Format | Example |
|---|---|---|
| Most GET responses (entity detail) | ISO 8601 with timezone | `2026-04-14T08:46:47+02:00` |
| Some CREATE responses | ISO 8601 without timezone | `2026-04-14T08:46:47` |
| List endpoints (`/task/{id}/subtasks`, `/all-projects`) | Space-separated, no `T`, no TZ | `2026-04-14 08:48:48` |

**Input**: `due_date` accepts `"2026-06-15"`, `"2026-06-15T00:00:00+02:00"`, and `"2026-06-15T06:59:57.535Z"`. The API canonicalizes internally.

### Currency format

Currency amounts are **strings**, value × 100, no separator.

| Amount | API value |
|--------|-----------|
| 1,000.25 | `"100025"` |
| 1,000 | `"100000"` |
| 1 | `"100"` |

Supported: `CZK`, `EUR`, `USD`.

### IDs — integers vs UUIDs

| Entity | ID type |
|---|---|
| Projects, tasklists, tasks (incl. subtasks as tasks), comments, notes, work reports, users, notifications, events, pinned items, invoices, **project-labels** | **integer** |
| **Task labels**, custom fields, custom-field enum options, files | **UUID** (string) |

### Priority

- Values: `"h"` (high), `"m"` (medium), `"l"` (low), `null` (none).
- **Field name is always `priority_enum`** on both CREATE and EDIT.
- The older field `priority` is silently ignored on CREATE — **always use `priority_enum`**.

### Colors — strict whitelist for labels

Freelo labels accept **only these 26 colors** (as hex). Any other value returns `400`:

```
#77787a  #15acc0  #367fee  #10aa40  #f2830b  #ca3e99  #9235e4  #e9483a
#ffffff  #e3b51e  #e8384f  #fd612c  #fda41a  #f4bd38  #a4c61a  #62d26f
#37a862  #159ddc  #4186e0  #7a6ff0  #aa62e3  #e362e3  #ea4e9d  #fc91ad
#8da3a6  #e9e9e9
```

If the user asks for an unsupported color, pick the closest match from the list.

### CREATE response vs. GET response

The response body from a `POST` create is **leaner** than the equivalent `GET` detail. For example, a task create response omits `state`, `cost`, `minutes`, `author`, `project`, `tasklist`, `custom_fields`, `estimates`, `date_edited_at`, `count_subtasks`, `parent_task_id`.

**After creating an entity you need to work with further, do a follow-up `GET` to see the full detail.**

### Author field — naming inconsistency

- Tasks, subtasks, comments, work reports: `author: {id, fullname}`
- Notes: `author: {id, name}` (singular `name`, not `fullname`)

When displaying an author name, try `fullname` first, fall back to `name`.

---

## Data Model

```
User (account)
├── Projects (int ID)           ── 3 listing endpoints: /projects, /all-projects, /invited-projects
│   ├── Workers                 (users assigned — may differ from owner)
│   ├── Project Labels          (int ID, color from whitelist) ── separate from task labels!
│   ├── Custom Fields           (UUID)
│   ├── Tasklists (int ID)
│   │   └── Tasks (int ID)
│   │       ├── Subtasks        (managed via /task/{task_id}/* — see Subtasks section)
│   │       ├── Comments (int ID)
│   │       ├── Description     (stored as a special comment, set via /task/{id}/description)
│   │       ├── Work Reports (int ID)
│   │       ├── Task Labels     (UUID, independent pool from project labels)
│   │       ├── Custom Field Values
│   │       ├── Files attachments (UUID)
│   │       ├── Reminders
│   │       ├── Estimates (total + per-user)
│   │       └── Public Link
│   ├── Notes (int ID)
│   ├── Pinned Items (int ID)
│   ├── Files & Documents (UUID)
│   └── Invoices (int ID)
│
├── Templates (project / tasklist / task)
├── Events (audit log)
└── Notifications
```

**State fields:**

- Projects: `1 = active`, `2 = archived`, `3 = template`.
- Tasks / subtasks: `state: {id: N, state: "<name>"}`. Known values:
  - `1` → `active`
  - `5` → `finished`
  - Other values seen in practice: `archived`, `deleted`, `template` — inspect the `state` name string when in doubt rather than relying on the numeric id.

---

## Common Patterns

### Resolving names → IDs

Users usually say "create a task in project Alpha" — you need the integer project ID:

1. Call `GET /projects` (and `GET /invited-projects` if the user is a collaborator, not the owner).
2. Match by name case-insensitively, accent-insensitively.
3. For tasks, use `POST /search` with `entity_type: "task"` — faster than listing.

### Drilling down

```
project → tasklists → tasks → subtasks/comments/work-reports
```

Fastest paths:

- All tasks in project X: `GET /all-tasks?projects_ids[]=X`
- Tasks in tasklist Y: `GET /project/{pid}/tasklist/Y/tasks` (bare array)
- All my active tasks: `GET /all-tasks` (paginated)

### After create, always GET

`POST` create responses are leaner than `GET` details. For downstream work, re-fetch:

```bash
# create
TASK_ID=$(curl -s -X POST ... /tasks | jq -r .id)
# get full detail
curl -s ... "$FREELO_BASE_URL/task/$TASK_ID"
```

### Confirm before destructive ops

For `DELETE`, `archive`, `force-delete` — echo the target name back to the user and confirm. **Deleted tasks cannot be recovered.**

---

## Response formatting — always link entities

When your response mentions a task, subtask, project, tasklist, comment, or note — by ID, name, or any form — **render it as a Markdown link to the Freelo web UI**. The user should be able to click any entity you mention. This is non-negotiable: do not force the user to ask "and what's the link?" after every answer.

### URL templates

| Entity | URL pattern |
|---|---|
| Project | `https://app.freelo.io/project/{id}/tasklists?layout=kanban` |
| Tasklist | `https://app.freelo.io/tasklist/{id}` |
| Task | `https://app.freelo.io/task/{id}` |
| Subtask | `https://app.freelo.io/task/{subtask_task_id}` — use the `task_id` field from the subtask response, NOT the `id` field (see Subtasks section) |
| Comment | `https://app.freelo.io/task/{task_id}#comment-{comment_id}` — both IDs required |
| Note | `https://app.freelo.io/note/{id}` |

### Good vs. bad example

**✅ Good** — clickable, informative:

> Found 3 overdue tasks:
> - **[Nabídka pro ABC](https://app.freelo.io/task/29234425)** — high priority, 5 days overdue
> - **[Reporty Q1](https://app.freelo.io/task/29234500)** — assigned to Anna
> - **[Podklady pro daňovku](https://app.freelo.io/task/29234600)** — high priority, 2 days overdue
>
> All in project **[Marketing Q1](https://app.freelo.io/project/591279/tasklists?layout=kanban)**.

**❌ Bad** — user has to ask for link, then click through Freelo UI manually:

> Found 3 overdue tasks: Nabídka pro ABC, Reporty Q1, Podklady pro daňovku.

### Rules

1. **Link text = human-readable name** of the entity (not the ID). If name is unknown, fall back to `Task #{id}` / `Project #{id}`.
2. **Every entity mention in the response gets a link** — including when listing multiple, quoting back an entity from the user's request, or confirming after a create/edit. If you mention the same task 3 times in a reply, link it every time (user reads linearly, not scanning).
3. **For subtasks** → use the `task_id` field from the API response (NOT `id`). The subtask-specific `id` is not routable in the web UI.
4. **For comments** → you need both `task_id` (the comment's parent task) and `comment_id` (from the create/edit response, or from the task's `comments[]` array).
5. **Do not link entities that don't have an ID yet** (e.g. a task you're about to create). Link it AFTER the create call returns with the ID.
6. **Projects** — the `/tasklists?layout=kanban` suffix is the canonical "main project view". If the user is looking for a specific sub-view (e.g. Calendar, Reports), you can drop the suffix and link just `/project/{id}/` — but default to the kanban URL.
7. **Do not link hypothetical entities** — only link what exists in the API data you fetched.

---

# Endpoint Reference

Organized by feature. Each section: **list → detail → create → update → delete** where applicable.

---

## Users & Workers

### Get current user

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/users/me"
```

Response: `{"result": "success", "user": {"id": 12345}}` — the minimal auth echo.

### List all coworkers

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/users"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {users: [...]}}`.

### List workers on a project

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}/workers"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {workers: [...]}}`.

### Invite users to projects

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"emails":["a@b.com","c@d.com"],"projects_ids":[123,456]}' \
  "$FREELO_BASE_URL/users/manage-workers"
```

### Remove workers from project

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"users_emails":["a@b.com","c@d.com"]}' \
  "$FREELO_BASE_URL/project/{project_id}/remove-workers/by-emails"
```

> **Body key is `users_emails`, not `emails`.** Sending `{"emails":[...]}` returns `400 Missing item 'users_emails'`. Note the asymmetry with the **invite** call (which uses `emails`) — they take different keys despite the parallel purpose.

---

## Projects

Freelo has **three project-listing endpoints** with different shapes and scopes — know when to use which:

### 1. `/projects` — projects where you are the **owner** (bare array)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/projects"
```

Bare array. Each element includes `tasklists` with nested task previews. **Does NOT include projects where you are only invited as a worker.**

### 2. `/all-projects` — same owner scope, but paginated

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-projects?p=0"
```

Response shape: `{total, count, page, per_page, data: {projects: [...]}}`.

### 3. `/invited-projects` — projects where you are a worker (invited)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/invited-projects?p=0"
```

Response shape: `{total, count, page, per_page, data: {invited_projects: [...]}}`.

> **When the user mentions a project and you cannot find it via `/projects`, also check `/invited-projects`.**

### Project detail

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
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
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-projects?state=archived"
```

### Template projects

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/template-projects"
```

Response shape: **paginated**, `{total, count, page, per_page, data: {template_projects: [...]}}`.

### Create project

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Project Name","currency_iso":"CZK"}' \
  "$FREELO_BASE_URL/projects"
```

`currency_iso` optional (default `CZK`). Values: `CZK`, `EUR`, `USD`.

### Archive / activate / delete project

```bash
# Archive
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}/archive"

# Reactivate
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}/activate"

# Delete (irreversible!)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}"
```

---

## Tasklists

### List all tasklists (paginated)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-tasklists"
```

Response: `{total, count, page, per_page, data: {tasklists: [...]}}`.

### List tasklists in a project

Use `GET /project/{id}` and read `.tasklists` from the response.

### Tasklist detail (includes tasks)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/tasklist/{tasklist_id}"
```

### Create tasklist

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Sprint 1"}' \
  "$FREELO_BASE_URL/project/{project_id}/tasklists"
```

Optional: `"budget": "100000"` (currency format, ×100).

> **Budget is write-only over the API.** The create response echoes `budget: {amount, currency}` once, but subsequent `GET /tasklist/{id}` and `GET /project/{id}` → tasklists[] responses **do not expose the budget field**. If the user needs to see a tasklist's current budget, direct them to the Freelo UI, or capture the value at create time.

### Edit tasklist — **not supported via API**

`POST /tasklist/{id}` returns 404. Tasklists are effectively immutable after creation — you cannot rename them, change their budget, or edit any other field through the public API. Changes must be made in the Freelo UI.

### Delete tasklist — **not supported via API**

There is no `DELETE /tasklist/{id}` endpoint (returns 404). If the user wants to remove a tasklist, ask them to delete it in the Freelo UI. As a workaround, you can move tasks out of it and leave it empty.

---

## Tasks

### List all tasks (paginated — the workhorse)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
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
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-tasks?states[]=finished&projects_ids[]={project_id}"

# Overdue — active tasks with due_date before today
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-tasks?states[]=active&due_date_to=2026-04-14"
```

> `/all-tasks/finished` and `/all-tasks/overdue` as separate paths return **404** — do not use them.

### Tasks in a tasklist

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}/tasklist/{tasklist_id}/tasks"
```

Response shape: **bare array**, not paginated.

### Task detail

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}"
```

Full detail — labels, author, worker, state, dates, cost, minutes, comments array, project, tasklist, estimates, custom fields.

> **`cost` and `minutes` are rolled-up**: calculated from the task's work reports (`minutes = sum of work-report minutes`, `cost = minutes × worker hourly rate`). You cannot set them directly on a task create/edit. A fresh task with no work reports has `cost: {amount: "0", currency: "CZK"}`, `minutes: 0`.

### Create task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{
    "name":"Fix login bug",
    "worker":12345,
    "due_date":"2026-06-15",
    "due_date_end":"2026-06-20",
    "priority_enum":"h",
    "comment":{"content":"Initial note"}
  }' \
  "$FREELO_BASE_URL/project/{project_id}/tasklist/{tasklist_id}/tasks"
```

**Body rules:**

- `worker` is an **integer** user ID. Passing `{"id": N}` returns HTTP 400.
- Priority field is `priority_enum`. The field `priority` is silently ignored on create.
- `due_date` and `due_date_end` accept `YYYY-MM-DD` or ISO 8601 with `+02:00` / `Z`.
- Optional `comment` seeds an initial comment.

### Edit task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New name","priority_enum":"m","worker":12345,"due_date":"2026-07-01","due_date_end":"2026-07-05"}' \
  "$FREELO_BASE_URL/task/{task_id}"
```

Send only fields you want to change.

> **Due-date range quirk:** When changing the end date, **always send both `due_date` AND `due_date_end` together**. Sending only `due_date_end` currently causes the API to overwrite `due_date` with the value and null out `due_date_end`. Always include the full range in edits.

### Finish / reactivate task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/finish"

curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/activate"
```

### Move task to another tasklist

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/move/{target_tasklist_id}"
```

### Delete task (irreversible)

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}"
```

Returns `{"result": "success"}`. **Prefer `finish` over `delete`** unless the user explicitly says "delete" — finish preserves history.

### Task description (HTML, stored as a comment)

The description is stored **internally as a special comment** that also shows up in the task's `comments[]` array. Use the dedicated endpoint:

```bash
# Get
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/description"

# Set / update — body key is "content", NOT "description"
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"<p>Full HTML content here</p>"}' \
  "$FREELO_BASE_URL/task/{task_id}/description"
```

Response includes `{id, content, date_add, files}` — the description has its own id and can have file attachments, just like a comment.

### Task description with file attachments

`POST /task/{id}/description` accepts a `files` array identical to comments:

```bash
# 1. Upload
FILE_UUID=$(curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -F "file=@./spec.pdf" "$FREELO_BASE_URL/file/upload" | jq -r .uuid)

# 2. Set description with file attached
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"<p>See attached spec</p>\",\"files\":[{\"uuid\":\"$FILE_UUID\"}]}" \
  "$FREELO_BASE_URL/task/{task_id}/description"
```

> **⚠️ Destructive quirk — `/description` can overwrite an existing comment!** If the task has no dedicated description yet but already has regular comments, calling `/description` may **replace the first comment's content and files with the new description payload**. Original comment data is lost.
>
> **Safe pattern**: set the description BEFORE adding regular comments, or create the task with `comment:{...}` in the create body (which marks the seed comment as description from the start). If the task may already have comments, GET the task first, inspect `comments[]` for an entry with `is_description: true`, and only call `/description` when you know what you are replacing.

### Task estimates — paid feature

Task time estimates are available only on paid Freelo plans. Accounts without the feature receive **HTTP 402 Payment Required** with `{"errors": ["Payment required. Your plan has been exceeded."]}`.

```bash
# Set total estimate (body key is `minutes`)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"minutes":480}' \
  "$FREELO_BASE_URL/task/{task_id}/total-time-estimate"

# Delete total estimate
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/total-time-estimate"
```

**Per-user estimates**: the public path is not yet confirmed (`/task/{id}/user-time-estimate` returns 404). If the user needs per-user estimates, recommend the Freelo UI or the OpenAPI docs until this path is validated.

The detail of an existing estimate appears in `GET /task/{id}` under `total_time_estimate` and `users_time_estimates[]`.

### Task reminders

```bash
# Create reminder
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"remind_at":"2026-06-15T09:00:00+02:00"}' \
  "$FREELO_BASE_URL/task/{task_id}/reminder"

# Delete reminder
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/reminder"
```

### Task public link — endpoint not currently reachable

`/task/{id}/public-link` returns 404 on both `GET` and `POST` in limited test contexts. This may be a paid feature or the public path has moved. If the user asks for a share link, point them to the Freelo UI "Share" action until this endpoint is validated against your plan.

---

## Subtasks

> **Important conceptual note:** In Freelo, a subtask is internally a **regular task with a parent task ID**. The `POST /task/{parent_id}/subtasks` endpoint returns an object with **two** ID fields:
>
> - `id` — a subtask-specific ID (exposed in UI, but **not** usable in further API calls)
> - `task_id` — the actual task ID. **Use this one for all subsequent operations** — detail, finish, activate, delete. They all go through `/task/{task_id}/*`, NOT `/subtask/{…}/*`.

### List subtasks of a task

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{task_id}/subtasks"
```

**Response shape — paginated**: `{total, count, page, per_page, data: {subtasks: [...]}}`.

Each subtask item has both `id` (subtask-specific) and `task_id` (usable task ID).

### Create subtask

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Write tests","worker":12345,"due_date":"2026-06-15","priority_enum":"m"}' \
  "$FREELO_BASE_URL/task/{parent_task_id}/subtasks"
```

Same body rules as tasks (worker = integer, priority_enum). **Read `task_id` from the response** for subsequent operations.

### Subtask operations — use `task_id`, not `id`

```bash
# Detail
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{subtask_task_id}"

# Finish
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{subtask_task_id}/finish"

# Reactivate
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{subtask_task_id}/activate"

# Delete
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/{subtask_task_id}"
```

> The endpoints `/subtask/{id}/finish`, `/subtask/{id}/activate`, `DELETE /subtask/{id}`, and `GET /subtask/{id}` all return **404** — they do not exist. Do not call them.

### Nested subtasks — **not supported, do not use**

Calling `POST /task/{subtask_task_id}/subtasks` to create a subtask underneath an existing subtask returns HTTP 200, **but the response has `"task_id": null`**:

```json
{"id": 16618517, "task_id": null, "name": "nested", ...}
```

Because `task_id` is null, the resulting "nested subtask" cannot be fetched, finished, activated, or deleted through `/task/{task_id}/*`. It is effectively orphaned in the API — only visible in the Freelo UI, if at all. **Do not create subtasks of subtasks via API.** If the user asks for a three-level hierarchy, either flatten it or advise them to use the Freelo web UI.

---

## Comments

### List all comments (paginated)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
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

### Create comment on a task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Looks good, merging now."}' \
  "$FREELO_BASE_URL/task/{task_id}/comments"
```

> **Quirk:** Plain text is automatically wrapped in `<div>...</div>` by the API. To embed real HTML, send it directly (e.g. `<p>Paragraph with <strong>bold</strong></p>`). The API stores rich HTML; plain text is a shorthand.

> **HTML sanitization (security)**: Freelo strips dangerous tags and attributes server-side. `<script>`, `<iframe>`, `onclick`, `onerror`, and similar XSS vectors are silently removed before storage. Example: sending `<p>Hi</p><script>alert(1)</script><p onclick=alert(1)>click</p>` ends up stored as `<p>Hi</p><p>click</p>`. Safe tags and attributes (p, ul, li, strong, em, a href, img src without event handlers, etc.) pass through unchanged. You can rely on user-supplied HTML being sanitized — but do not trust the reverse (assume anything round-trips exactly).

> **HTML auto-balancing**: Unclosed or mis-nested tags are auto-closed in the correct order before storage. Example: `<p>unclosed <strong>bold <em>italic` becomes `<p>unclosed <strong>bold <em>italic</em></strong></p>`. You don't need to perfectly close your HTML — the API will fix it for you. Still, prefer sending well-formed HTML where possible so the stored output matches your intent.

### Edit comment

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated text"}' \
  "$FREELO_BASE_URL/comment/{comment_id}"
```

**Attach new files to an existing comment** — the `files` field works on edit too:

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated with file","files":[{"uuid":"abc-1234-..."}]}' \
  "$FREELO_BASE_URL/comment/{comment_id}"
```

Existing files on the comment are preserved; the new UUIDs are added. Use this to retro-attach a file to a comment you already posted.

### Delete comment — **not supported via API**

The REST API has no delete endpoint for comments. `DELETE /comment/{id}`, `/comments/{id}`, `/comment/{id}/delete` — all return 404. If the user wants to remove a comment, ask them to delete it through the Freelo web UI, or edit the content to empty text as a workaround.

---

## Labels — **two separate systems**

Freelo has **two independent label entities** that are easy to confuse:

| | Project Labels | Task Labels |
|---|---|---|
| ID type | **integer** | **UUID** |
| Scope | Defined per project | Global pool (shared across tasks) |
| Colors | From whitelist only (see API Basics) | From whitelist only |
| How to use | Set up at project level | Apply directly to a task |

> **Important:** Adding a label "by name" to a task **does not** link to a project label with the same name — it creates a **new task label** in the global pool with default color `#77787a` (grey). The two systems are decoupled.

### List available project labels

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project-labels/find-available"
```

Response shape: `{"labels": [...]}` — a dict with a `labels` key, not a bare array. Each label has `{id (int), name, color, is_private, users_id, usage_count, can_be_public, can_be_edited}`.

### Add a label to a project

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Priority","color":"#e9483a","is_private":false}' \
  "$FREELO_BASE_URL/project-labels/add-to-project/{project_id}"
```

All three fields are required: `name`, `color` (from whitelist), `is_private` (bool). Response: `{"result": "success"}`.

### Edit a project label

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New name","color":"#e9483a","is_private":false}' \
  "$FREELO_BASE_URL/project-labels/{label_id}"
```

### Delete a project label (cleanest removal)

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project-labels/{label_id}"
```

Returns `{"result": "success"}`. **Prefer this over `/project-labels/remove-from-project/...`** — that endpoint currently requires extra fields and is unreliable.

### Task labels — global pool

```bash
# Create one or more task-labels (they go into the global pool).
# Body MUST be wrapped in a "labels" array — sending {"name":..., "color":...} at the top level returns 400.
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"name":"Bug","color":"#e9483a"}]}' \
  "$FREELO_BASE_URL/task-labels"
```

Response: `{"result": "success"}`. The created labels do not come back in the body — re-fetch via `/task-labels/add-to-task/{tid}` response (which includes the UUID) or by adding the label to a task and reading the task's `labels[]`.

### Add task label(s) to a task

```bash
# By UUID (preferred — uses an existing task label)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"uuid":"abc-uuid-here"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"

# By name (creates a NEW task label in the global pool with default grey color)
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"name":"Bug"}]}' \
  "$FREELO_BASE_URL/task-labels/add-to-task/{task_id}"
```

### Remove task label from a task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"labels":[{"uuid":"abc-uuid-here"}]}' \
  "$FREELO_BASE_URL/task-labels/remove-from-task/{task_id}"
```

Response: `{"result": "success"}`.

---

## Custom Fields

Custom fields are project-scoped and use UUIDs. Values are set per task.

### List custom field types

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field/get-types"
```

Response shape: `{"custom_field_types": [{"uuid": "...", "name": "..."}]}`.

**Seven supported types** (names as returned by API):

- `number`
- `text`
- `date`
- `date_time`
- `bool`
- `link`
- `enum`

### List fields in a project

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field/find-by-project/{project_id}"
```

Response shape: `{"custom_fields": [...], "is_commander": bool}`. The `is_commander` flag tells you whether the current user has permission to edit/delete custom fields in this project.

### Create custom field

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Story Points","type":"<type-uuid-from-get-types>"}' \
  "$FREELO_BASE_URL/custom-field/create/{project_id}"
```

> **Body key is `type`, not `type_uuid`.** Sending `type_uuid` returns `400: Parameter type not found.`

### Rename / delete / restore field

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Estimate"}' \
  "$FREELO_BASE_URL/custom-field/rename/{field_uuid}"

curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field/delete/{field_uuid}"

curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field/restore/{field_uuid}"
```

### Set / delete value on a task (text / number fields)

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"custom_field_uuid":"abc-uuid","value":"42"}' \
  "$FREELO_BASE_URL/custom-field/add-or-edit-value"

curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field/delete-value/{value_uuid}"
```

### Enum options (for fields of type `enum`)

```bash
# List options for a field
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field-enum/get-for-custom-field/{field_uuid}"

# Create option
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Option A","color":"#e9483a"}' \
  "$FREELO_BASE_URL/custom-field-enum/create/{field_uuid}"

# Edit option
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Option B","color":"#62d26f"}' \
  "$FREELO_BASE_URL/custom-field-enum/change/{enum_uuid}"

# Delete option (safe, fails if used on tasks)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field-enum/delete/{enum_uuid}"

# Force-delete (removes from all tasks too)
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/custom-field-enum/force-delete/{enum_uuid}"

# Set enum value on a task
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"custom_field_uuid":"abc-uuid","custom_field_enum_uuid":"def-uuid"}' \
  "$FREELO_BASE_URL/custom-field/add-or-edit-enum-value"
```

---

## Notes

### Create note in a project

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Retro notes","content":"<p>HTML content</p>"}' \
  "$FREELO_BASE_URL/project/{project_id}/note"
```

### Get / edit / delete note

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/note/{note_id}"

curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New title","content":"<p>New content</p>"}' \
  "$FREELO_BASE_URL/note/{note_id}"

curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/note/{note_id}"
```

> Note the naming inconsistency: the note's author has `author: {id, name}` (singular `name`), whereas tasks/comments use `{id, fullname}`.

> **HTML sanitization on notes is stricter than on comments.** Notes strip heading tags (`<h1>` through `<h6>`) before storage — the text content is kept, but the wrapping tags are removed. Safe tags like `<p>`, `<ul>`, `<li>`, `<strong>`, `<em>` pass through. The entire content is also wrapped in an outer `<div>`. Example: input `<h3>Title</h3><p>body</p>` becomes `<div>Title<p>body</p></div>`. If you need structural hierarchy, use `<strong>` or `<p>` with bold rather than heading tags.

---

## Time Tracking

Only **one** timer runs per user at a time. Starting a new one while another is active returns HTTP 409.

### Start tracking

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":12345,"note":"Implementing feature X"}' \
  "$FREELO_BASE_URL/timetracking/start"
```

Both fields optional — you can track "free" time without a task. Response is minimal: `{"uuid": "..."}`.

### Status (the rich view)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/timetracking/status"
```

Returns the full active session: `{uuid, date_reported, task: {id, name, project, tasklist}, note, cost, is_billable, project_setting, ...}`.

### Stop (auto-creates work report if task was set)

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/timetracking/stop"
```

Returns the created work report (when a task was attached): `{id, minutes, note, task, author, worker, cost, date_reported, date_add}`.

### Edit an active tracking session

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"task_id":67890,"note":"Switched focus"}' \
  "$FREELO_BASE_URL/timetracking/edit"
```

---

## Work Reports

### Create work report manually

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"minutes":120,"date_reported":"2026-06-15","note":"Implementation","worker_id":12345}' \
  "$FREELO_BASE_URL/task/{task_id}/work-reports"
```

- `minutes` — required.
- `date_reported` — required, `YYYY-MM-DD`.
- `note`, `worker_id` — optional. If `worker_id` is omitted, the report is attributed to the current user.

### Edit / delete work report

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"minutes":90,"note":"Adjusted"}' \
  "$FREELO_BASE_URL/work-reports/{report_id}"

curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/work-reports/{report_id}"
```

### List work reports — known limitation

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/work-reports?projects_ids[]={project_id}&users_ids[]={user_id}&p=0"
```

Response shape: `{total, count, page, per_page, data: {reports: [...]}}`.

> **Caveat:** in limited-scope API contexts the listing may return `total: 0` even when reports exist. If this happens, fetch individual reports via `/work-reports/{report_id}` using IDs you have from create/timer-stop responses, or inspect a task's detail for its rolled-up `minutes`.

---

## Files & Documents

Files in Freelo have a specific three-step lifecycle — **upload, attach, then access**. Skipping the attach step leaves the file in an "orphan" state where it exists but cannot be downloaded.

### The file workflow

```
┌──────────┐     ┌──────────────────────┐     ┌───────────────┐
│ 1. Upload│ ──► │ 2. Attach to entity  │ ──► │ 3. Download   │
│ → uuid   │     │ (comment/note/etc.)  │     │ /file/{uuid}  │
└──────────┘     └──────────────────────┘     └───────────────┘
                          ▲
                          │
                     Until attached,
                   /file/{uuid} returns
                 "This resource does not exists"
```

### List files in a project

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-docs-and-files?projects_ids[]={project_id}"
```

Filter by `?type=file` (or `directory`, `link`, `document`).
Response shape: `{total, count, page, per_page, data: {items: [...]}}` — note the `items` key.

### Step 1 — Upload a file

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -F "file=@./local-file.pdf" \
  "$FREELO_BASE_URL/file/upload"
```

- Max size: **100 MB**.
- Send the file via multipart form-data (`-F "file=@..."`).
- **One file per upload call.** Passing two `-F file=@...` parameters in a single request returns only **one** UUID — the second file is ignored. Upload files sequentially, one request per file.
- **Response is minimal**: `{"uuid": "abc-1234-..."}` — just the UUID, nothing else. Capture it.
- **Do NOT rely on a filename in the response** — there is none. You already know the filename locally.
- **Zero-byte (empty) files are accepted** without error — useful for placeholder uploads, but confirm with the user if this is intentional.
- **Each UUID is a one-shot claim token**: it can be attached to exactly **one** entity. Re-attaching the same UUID to a second comment returns HTTP 200 with `files:[]` — a silent drop. Upload a fresh file if you need to attach it twice.

### Step 2 — Attach the uploaded file to an entity

The UUID from upload is a **claim token**: it's not bound to any project/task until you attach it. Attach via the entity's body using a `files` array.

**Files array shape** — **array of objects, not strings**:

```json
"files": [ { "uuid": "abc-1234-..." }, { "uuid": "def-5678-..." } ]
```

> `"files": ["abc-1234-..."]` (array of plain strings) returns `400 Every items of $files must be array`. Always wrap in an object.

#### Attach to a new task comment

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"See attached","files":[{"uuid":"abc-1234-..."}]}' \
  "$FREELO_BASE_URL/task/{task_id}/comments"
```

#### Attach to an initial comment on task create

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Task","worker":12345,"comment":{"content":"With file","files":[{"uuid":"abc-1234-..."}]}}' \
  "$FREELO_BASE_URL/project/{pid}/tasklist/{tlid}/tasks"
```

#### Notes — **file attachment not supported via API**

Adding `"files":[{"uuid":"..."}]` to a `POST /project/{pid}/note` or to `POST /note/{id}` (edit) returns HTTP 200 but the `files` array on the note stays empty — the UUIDs are silently dropped. Notes do not currently accept file attachments through the public API. If the user needs files on a note, attach them to a task instead, or ask the user to upload via the Freelo web UI.

### Step 3 — Download a file

Once the file has been attached to any entity, download via the UUID:

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -o local-file.pdf \
  "$FREELO_BASE_URL/file/{file_uuid}"
```

The response is the raw file bytes — save with `-o`. **Auth is mandatory** on every file endpoint — a missing `Authorization` header returns HTTP 401 (`{"errors":["HTTP Authorization header was not found."]}`).

> **If you get `{"errors":["This resource does not exists."]}`** on download, the file has not been attached yet. Attach it first (Step 2), or the UUID is simply wrong.

### Valid attachment targets — and what does NOT accept files

| Target | Works? | How |
|---|---|---|
| Task comment (create) | ✅ | `POST /task/{id}/comments` with `files:[{uuid}]` |
| Task comment (edit) | ✅ | `POST /comment/{id}` with `files:[{uuid}]` (adds to existing) |
| Task seed comment during task create | ✅ | `comment:{content,files:[...]}` in task create body |
| Task description | ✅ | `POST /task/{id}/description` with `files:[{uuid}]` — but see destructive-overwrite warning in the Tasks section |
| **Task itself** (task body) | ❌ | `POST /task/{id}` with `files:[...]` returns 200 but silently drops the UUIDs — tasks have no `files` field. Attach to the task's comment or description instead. |
| **Notes** | ❌ | `files` on `POST /project/{pid}/note` / `POST /note/{id}` is silently ignored. Not supported. |
| **Subtasks** | Use the subtask's (task_id) comments — same pattern as tasks |

### File persistence after detach or task deletion

> **Files live independently of attachments.** Removing a file from a comment (editing with `files:[]`) or deleting the owning task does **not** delete the underlying file — `GET /file/{uuid}` still returns the bytes to anyone authenticated who knows the UUID.
>
> **Privacy implication:** do not assume "deleting the task" removes the file. If a sensitive file was accidentally uploaded, there is no public API for truly deleting it — direct the user to the Freelo web UI or support.

### Server-side filename sanitization

The `filename` in the response is **not** the name you uploaded — it is rewritten by the server:

- Prefix: random hash + user id (e.g. `hsjn360xu12345-`).
- Spaces in the original filename are replaced with dashes (`r7 with spaces.txt` → `r7-with-spaces.txt`).
- Some diacritics and special characters may also be transliterated.

**Always keep the original filename client-side** if you need to surface it back to the user. Do not parse the server `filename`.

### File metadata on an attached entity — two shapes

Depending on where you read the file info, Freelo returns a **leaner or richer** shape. Be ready for both.

**Leaner** — immediate response from `POST /task/{id}/comments` (comment create):

```json
{
  "id": 12889534,              // internal integer id
  "filename": "6vk87o1wu12345-report.pdf",
  "size": 44                   // bytes
}
```

**Richer** — same file read via `GET /task/{id}` inside `comments[].files`:

```json
{
  "uuid": "57771de9-be71-4edf-9168-b476f6990bcd",   // the upload UUID
  "filename": "msmfqv9su12345-notes.txt",
  "caption": null,             // may be set in the Freelo UI
  "description": null,         // may be set in the Freelo UI
  "date_add": "2026-04-14T09:48:07+02:00",
  "date_edited_at": null,
  "size": 26
}
```

**Key takeaways:**

- The `uuid` is what you need for download (`GET /file/{uuid}`). It is present only in the richer shape.
- The `id` in the leaner shape is an internal int — do **not** use it for download.
- `filename` is **server-prefixed** (random hash + user ID) in both shapes. Never surface it as a user-facing display name — preserve the original filename from the upload input.
- `caption` and `description` come from the Freelo UI's "file info" editor; expect `null` when files are attached via API.

### Full upload-and-attach example (copy-paste)

```bash
# 1. Upload
UPLOAD_RESP=$(curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -F "file=@./report.pdf" "$FREELO_BASE_URL/file/upload")
FILE_UUID=$(echo "$UPLOAD_RESP" | jq -r .uuid)

# 2. Attach to comment on task 12345
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"Attached report\",\"files\":[{\"uuid\":\"$FILE_UUID\"}]}" \
  "$FREELO_BASE_URL/task/12345/comments"

# 3. Download it back (as a new requester would)
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -o ./downloaded.pdf "$FREELO_BASE_URL/file/$FILE_UUID"
```

---

## Templates

### List template projects

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/template-projects"
```

Paginated shape: `{data: {template_projects: [...]}, ...}`.

### Create from template

```bash
# Project from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New Project"}' \
  "$FREELO_BASE_URL/project/create-from-template/{template_id}"

# Tasklist from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/tasklist/create-from-template/{template_id}"

# Task from template
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/task/create-from-template/{template_id}"
```

---

## Pinned Items

```bash
# List
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/project/{project_id}/pinned-items"

# Create
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"link":"https://example.com/doc","title":"Design spec"}' \
  "$FREELO_BASE_URL/project/{project_id}/pinned-items"

# Delete
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/pinned-item/{pinned_id}"
```

---

## Notifications

```bash
# List all
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-notifications"

# Unread only
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/all-notifications?unread=1"

# Mark read / unread
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/notification/{notification_id}/mark-as-read"

curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/notification/{notification_id}/mark-as-unread"
```

---

## Events (audit log)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
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
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/events?date_from=2026-03-01&date_to=2026-04-01"
```

Each event is rich — includes `author` (with out-of-office info), nested `project`, `task`, `tasklist`, `comment`, `file`, `document` references (any irrelevant ones are `null`), and a `type` string like `new_project`, `new_task`, `finish_task`, etc.

---

## Out of Office

### Status

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```

Response when inactive: `{"out_of_office": null}`.
Response when active: `{"out_of_office": {"date_from": "...", "date_to": "..."}}`.

### Enable

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"out_of_office":{"date_from":"2026-07-01 00:00:00","date_to":"2026-07-14 23:59:59"}}' \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```

**Body rules:**

- Wrap the dates in an `out_of_office` object — sending `{"from": ..., "to": ...}` or top-level `{"date_from": ..., "date_to": ...}` both return 400.
- Keys are `date_from` / `date_to`, **not** `from` / `to`.
- Input format: space-separated local Prague time (`YYYY-MM-DD HH:MM:SS`).
- **Timezone conversion quirk**: input is interpreted as Europe/Prague local time, but the `GET` response returns them converted to UTC with `Z` suffix (e.g. input `2026-07-01 00:00:00` → response `2026-06-30T22:00:00Z` in summer). Expect the shift of `+02:00` / `+01:00`.

### Disable

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```

---

## Invoices

```bash
# List issued invoices
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/issued-invoices?projects_ids[]={project_id}"

# Invoice detail
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  "$FREELO_BASE_URL/issued-invoice/{invoice_id}"

# Mark as invoiced
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://accounting.com/inv-123","subject":"Invoice #123"}' \
  "$FREELO_BASE_URL/issued-invoice/{invoice_id}/mark-as-invoiced"
```

---

## Search

The single most useful endpoint for resolving names to IDs.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.0" \
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

---

# Troubleshooting & Error Handling

### HTTP status codes

| Code | Meaning | What to do |
|---|---|---|
| 200 / 201 | Success | Parse response body |
| 204 | Success, no content | No body |
| 400 | Bad request | Read `errors[]` — usually field format/validation |
| 401 | Unauthorized | Credentials wrong — re-verify with `/users/me` |
| 402 | Payment required | Feature gated by the account's paid plan (estimates, some custom-field types, premium endpoints). Error body: `{"errors":["Payment required. Your plan has been exceeded."]}`. Surface it verbatim — it's an account limit, not a request error. |
| 403 | Forbidden | User lacks access to this resource |
| 404 | Not found | ID does not exist, endpoint path is wrong, or feature not on this plan |
| 409 | Conflict | E.g., starting a second timer while one is running |
| 422 | Unprocessable | Body parsed but business rule violated |
| 429 | Rate limit | Wait 60 seconds, then retry |
| 5xx | Server error | Retry once after a few seconds; if persistent, report |

### Error shape — **two patterns**

```json
// 400 / 422
{ "errors": ["Field 'name' is required.", "Color is not a valid value."] }

// 404 / 401 (sometimes)
{ "error": "Page not found, HTTP code 404." }
```

When parsing, check **both** `errors` (array) and `error` (string).

### Common pitfalls

- **`worker` as object on CREATE** → HTTP 400. Use integer: `"worker": 12345`.
- **`priority` instead of `priority_enum`** → silently ignored, task has no priority. Always use `priority_enum`.
- **Body key `description` for `/task/{id}/description`** → 400. The key is `content`.
- **Single quotes around headers with `$VARS`** → variable not expanded.
- **`/subtask/{id}/finish`**, `/activate`, `DELETE /subtask/{id}`, `GET /subtask/{id}` — all 404. Subtasks live under `/task/{task_id}/…`. Use the `task_id` field from the subtask response, not the `id` field.
- **Bare-array endpoints** have **no pagination** — no `total`, no `?p=N`. Do not attempt `data.tasks` on them.
- **Color outside the whitelist** on label create → 400.
- **`add-to-project` for labels without `is_private`** → 400.
- **Sending only `due_date_end` in a task edit** → the API overwrites `due_date` with it and nulls `due_date_end`. Send both fields together.
- **Timer already running** when calling `/timetracking/start` → 409. Call `/timetracking/stop` or `/timetracking/edit` first.
- **`DELETE /comment/{id}`** → not supported. Edit the comment to empty text as a workaround.
- **`/projects` missing a project** → you are probably only invited. Try `/invited-projects` or `GET /project/{id}` directly.
- **`/work-reports` returning `total: 0`** despite reports existing → may be a scope limitation in limited API contexts.
- **Create custom field with `type_uuid`** → 400 (`Parameter type not found.`). The correct body key is `type`.
- **Plan limit exceeded** → HTTP 402 with `{"errors":["Payment required. Your plan has been exceeded."]}`. Surface verbatim; it's an account-level constraint, not a request error.
- **Total estimate body key** → use `minutes`, not `estimate_minutes`. The older name is silently rejected.
- **Separate `/all-tasks/finished`, `/all-tasks/overdue`** → 404. Use `GET /all-tasks?states[]=finished` or filter by `due_date_to` for overdue.
- **Task public link (`/task/{id}/public-link`)** → 404 on limited plans; not universally available. Fall back to the Freelo UI.
- **OOO body without `out_of_office` wrapper** → 400 (`Missing item 'out_of_office'`). Body must be `{"out_of_office": {"date_from": "...", "date_to": "..."}}`. Keys are `date_from` / `date_to`, not `from` / `to`.
- **`POST /task-labels` top-level name/color** → 400 (`Missing item 'labels'`). Body must be wrapped: `{"labels":[{"name":"...","color":"..."}]}`.
- **`DELETE /tasklist/{id}`** → 404. Tasklists cannot be deleted via API; direct the user to the web UI.
- **`/all-docs-and-files` data key** → `items`, not `docs_and_files`.
- **File download returns `"This resource does not exists"`** → the file was uploaded but never attached. Attach it via `"files":[{"uuid":"..."}]` on a comment/note/task-create, then the download works.
- **File attachment `"files":["uuid-string"]`** → 400. Must be `"files":[{"uuid":"..."}]` — array of objects, not strings.
- **File upload response lacking filename** → by design; upload returns only `{uuid}`. Preserve the original filename client-side.
- **Search result `state` is an integer**, not the usual `{id, state}` object — do not try to read `.state.id` on search items.
- **Re-attaching the same file UUID** returns HTTP 200 but `files:[]` — the UUID was silently dropped. Each uploaded UUID is a one-shot claim; upload a fresh copy if you need to attach the file twice.
- **Multi-file upload in one request** ignores all but the first file. Upload one file per `/file/upload` call, then attach them together in a single comment's `files` array if you need them grouped.
- **Files on notes via API** — not supported. The `files` field on `POST /project/{pid}/note` / `POST /note/{id}` is silently ignored. Attach files to a task comment instead.
- **Files on task body** — not supported. `POST /task/{id}` with `files:[...]` is silently ignored. Attach to the task's description or a comment instead.
- **`/task/{id}/description` overwrote a comment's content** — calling `/description` on a task whose first comment is not yet flagged as description can replace that first comment. Always check `comments[]` for `is_description: true` before setting description on a task that already has comments.
- **File still downloadable after task delete / detach** — files live independently of their attachments. `files:[]` edit or `DELETE /task/{id}` does not remove the file bytes. There is no public endpoint to permanently delete an uploaded file.
- **Detaching files** — to remove all files from a comment without deleting the comment, edit it with `files:[]`: `POST /comment/{id}` with `{"content":"...","files":[]}` preserves content but clears attachments.
- **Server filename rewriting** — do not rely on the server-returned filename. It is prefixed with a random hash and sanitized (spaces → dashes). Preserve the original filename client-side.
- **File endpoint auth** — `/file/upload` and `/file/{uuid}` both require `Authorization` header. Missing auth → HTTP 401.
- **Nested subtask returns `task_id: null`** — Freelo does not support subtasks of subtasks via API. The create call succeeds with HTTP 200 but leaves you with an unusable record. Flatten the hierarchy or refer the user to the web UI.
- **Search 1-char or empty query** → 400. Require at least 2 characters before calling `/search`.
- **Worker remove body key** → `users_emails` (plural, prefixed). The parallel invite call uses plain `emails`. They are inconsistent — do not assume one from the other.
- **Tasklist edit / delete** → no API. Tasklists are write-once; only `name` and `budget` can be set at create time. For renames or removal, direct the user to the Freelo UI.
- **Tasklist `budget` disappears after create** — the create response shows it once; later GETs do not include it. Capture the value at create time if you need it later.
- **Task `cost` or `minutes` sent in create/edit body** → silently ignored. These are computed rollups from work reports + hourly rates, not inputs.
- **Heading tags (`<h1>`..`<h6>`) in note content** → stripped by the stricter notes sanitizer; only the text remains. Use `<p>` + `<strong>` for visual headings inside notes.

---

# Tips for Fast, Accurate Work

1. **Start from `GET /projects`** when the user mentions a project by name. If not found, try `GET /invited-projects` — you may be a collaborator, not owner.

2. **`/all-tasks` is the workhorse** — use its query filters (`projects_ids[]`, `states[]`, `q`, `due_date_from`) instead of drilling down manually.

3. **Use `POST /search`** to resolve task / comment names to IDs — faster and more flexible than listing.

4. **After every `POST` create, issue a follow-up `GET`** if you need the full entity. Create responses are lean.

5. **Always confirm before destructive actions** (`DELETE`, archive, force-delete) — echo the target name + ID to the user first.

**Always render entity mentions as clickable Markdown links** to the Freelo web UI — never force the user to ask "what's the link?". See the Response formatting section above for URL templates. This applies to every task, subtask, project, tasklist, comment, and note the response mentions.

6. **Prefer `finish` over `delete` for tasks** unless the user explicitly says "delete" — finish preserves history.

7. **Batch-safe iteration**: when paginating, start at `p=0`, increment until `total <= (page+1) * per_page`. Do not parallelize across pages — rate limit is 25/min.

8. **Be explicit about HTML** — comments and task descriptions store HTML. Plain text in a comment is auto-wrapped in `<div>`; to get `<p>`, `<ul>`, etc., send them yourself.

9. **Never pick an arbitrary label color** — use only the 26 colors from the whitelist (see API Basics). If the user requests a different hex, pick the closest one.

10. **Respect the colon**: `comment: null` means "no initial comment", an empty string `""` is different. Freelo distinguishes `null` from empty.

11. **Czech context**: Freelo is Czech-first — project/task names are often in Czech. Search is accent-insensitive for Czech characters.

12. **Cache per conversation**: projects list, workers list, custom field definitions, label whitelist — fetch once and reuse within a conversation.

13. **Dates in edit** — when editing a date range (`due_date` + `due_date_end`), send both. Sending only the end date silently breaks the range.

14. **Two label systems** — project labels (int, per project) and task labels (UUID, global pool) are separate. Adding "by name" to a task does not link to a project label with that name.

15. **When something fails**, check the full error body. Freelo's error messages are usually specific and actionable.

16. **Prod vs. dev environments**: `$FREELO_BASE_URL` can point to a staging instance — confirm you are in prod before destructive actions.

---

# Fallback: Full API Reference

For endpoints and fields not covered here, use the live OpenAPI documentation:

**<https://api.freelo.io/docs/v1/freelo-api>**

Use `WebFetch` to retrieve specific sections when needed. The endpoints above cover the common **95 %** of Freelo operations.
