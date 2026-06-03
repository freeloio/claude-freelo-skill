# Curl conventions

Load when writing any new curl call, decoding an error, paginating, or working with dates/currency/IDs/priority/colors.

## Contents

- API Basics
- Data Model
- Common Patterns
- Troubleshooting & Error Handling
- Tips for Fast, Accurate Work

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
