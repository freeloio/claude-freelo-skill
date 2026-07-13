---
name: freelo
description: Use when interacting with the Freelo.io project management API — managing projects, tasklists, tasks, subtasks, comments, time tracking, work reports, labels, custom fields, notes, files, templates, pinned items, workers, notifications, events, invoices, or searching in Freelo. Trigger this skill whenever the user mentions Freelo, wants to manage tasks or projects in Freelo, track time, create work reports, check invoices, or do anything related to Freelo.io — even if they do not explicitly say "API".
---

# Freelo API

Freelo.io is a project and task management tool. This skill calls Freelo's REST API. **API reference:** <https://api.freelo.io/docs/v1/freelo-api>.

> 🔗 **Critical formatting rule — read before composing any reply.**
> Whenever you mention a Freelo entity (task, subtask, project, tasklist, comment, note) — in prose, lists, or table cells — render it as `[name](url)`, never as a bare URL. A bare URL shows the user the URL, not the entity name; they cannot tell which entity it is without clicking. **Especially in tables**: the entity column must contain `[Task name](https://app.freelo.io/task/12345)`, NOT `https://app.freelo.io/task/12345`. Full rules, examples, and the terminal-renderer fallback are in `references/response-formatting.md`.

## Authentication

HTTP Basic Auth with credentials from environment variables (configured in `~/.claude/settings.json`):

- **Username**: `$FREELO_EMAIL` · **Password**: `$FREELO_API_KEY` · **Base URL**: `$FREELO_BASE_URL` (default: `https://api.freelo.io/v1`)
- Every request **must** include `User-Agent: Freelo-Claude-Skill/1.1.0` — Freelo identifies skill traffic by this exact value for adoption telemetry. Do not substitute.

If `$FREELO_EMAIL` or `$FREELO_API_KEY` is unset, stop and tell the user to add them to `~/.claude/settings.json` and restart Claude Code. Do not ask for credentials in the conversation.

```bash
FREELO_BASE_URL="${FREELO_BASE_URL:-https://api.freelo.io/v1}"

curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" \
  -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  "$FREELO_BASE_URL/{endpoint}"
```

For POST/PUT add `-d '{"key": "value"}'`. Use **double quotes** around headers containing shell variables, single quotes around the JSON body. Verify credentials with `GET /users/me` → returns `{"result": "success", "user": {...}}`.

## URL templates for response links

Always available — wrap every entity mention as `[name](url)`:

| Entity | URL pattern |
|---|---|
| Project | `https://app.freelo.io/project/{id}/tasklists?layout=kanban` |
| Tasklist | `https://app.freelo.io/tasklist/{id}` |
| Task | `https://app.freelo.io/task/{id}` |
| Subtask | `https://app.freelo.io/task/{subtask_task_id}` — use `task_id`, NOT `id` |
| Comment | `https://app.freelo.io/task/{task_id}#comment-{comment_id}` |
| Note | `https://app.freelo.io/note/{id}` |

## Curl conventions (summary)

Rate limit **25 req/min**. Errors: `{"errors":[...]}` (400/422) or `{"error":"..."}` (404/401) — check both. Pagination: `?p=N` (0-indexed); `/search` uses `{"page":N}` in body. Full rules, response shapes, dates, currency, ID types, priority, color whitelist, HTTP codes, and ~35-bullet pitfalls list → **[references/curl-conventions.md](references/curl-conventions.md)** (load when writing a new curl call or decoding an error).

## Data model (summary)

`Project (int)` → `Tasklist (int)` → `Task (int)` → `Subtask` (managed via `/task/{task_id}/*`, use `task_id` NOT `id`) / `Comment (int)` / `Work Report (int)` / `Task Label (UUID)` / `Custom Field Value` / `File (UUID)`. Project also owns `Note (int)`, `Pinned Item (int)`, `Project Label (int)`, `Custom Field (UUID)`, `Invoice (int)`. State: projects `1=active 2=archived 3=template`; tasks/subtasks `1=active 5=finished`. Full breakdown in `references/curl-conventions.md`.

## Workflow

1. Find the user's intent in the references table below.
2. Load **only** the matching reference. References do not chain — they are one level deep.
3. Compose the reply with every entity wrapped as `[name](url)` using the templates above.
4. Confirm destructive ops (delete, archive, force-delete) by echoing the target name back to the user before calling.
5. After a `POST` create, follow up with `GET` if you need full detail — create responses are leaner than `GET` responses.

## References

| Reference | When to load |
|---|---|
| **Shared** | |
| [curl-conventions.md](references/curl-conventions.md) | Writing any new curl call; decoding errors; pagination; dates/currency/IDs/priority/colors; HTTP codes; pitfalls |
| [response-formatting.md](references/response-formatting.md) | Composing any reply that mentions a Freelo entity; user reports "I just see URLs, where are the names?" |
| **Users & Workers** | |
| [get-current-user.md](references/get-current-user.md) | "Who am I" / verify credentials / get my user ID |
| [list-all-coworkers.md](references/list-all-coworkers.md) | "List all coworkers" / show users on my account |
| [list-project-workers.md](references/list-project-workers.md) | "Who is on project X" |
| [invite-users-to-projects.md](references/invite-users-to-projects.md) | "Invite X to project Y" |
| [remove-workers-from-project.md](references/remove-workers-from-project.md) | "Remove X from project Y" |
| **Projects** | |
| [list-projects.md](references/list-projects.md) | "List my projects" / all / archived / templates / "I can't find project X" |
| [get-project-detail.md](references/get-project-detail.md) | "Show project X details" / budget / time rollups |
| [create-project.md](references/create-project.md) | "Create a project" |
| [archive-project.md](references/archive-project.md) | "Archive project X" |
| [activate-project.md](references/activate-project.md) | "Reactivate / unarchive project X" |
| [delete-project.md](references/delete-project.md) | "Delete project X" (irreversible — confirm first) |
| **Tasklists** | |
| [list-tasklists.md](references/list-tasklists.md) | "List tasklists in project X" |
| [create-tasklist.md](references/create-tasklist.md) | "Create a tasklist" |
| [edit-or-delete-tasklist-not-supported.md](references/edit-or-delete-tasklist-not-supported.md) | User asks to rename / edit / delete a tasklist (UI-only) |
| **Tasks** | |
| [list-tasks.md](references/list-tasks.md) | "List tasks" / today / overdue / finished / by project / tasklist / worker / label / text query |
| [get-task-detail.md](references/get-task-detail.md) | "Show task X details" / "status of task Y" |
| [create-task.md](references/create-task.md) | "Create a task" |
| [edit-task.md](references/edit-task.md) | "Rename task" / change priority / reassign / change dates |
| [finish-task.md](references/finish-task.md) | "Finish / complete / close task X" |
| [reactivate-task.md](references/reactivate-task.md) | "Reopen task X" |
| [move-task.md](references/move-task.md) | "Move task X to tasklist Y" |
| [delete-task.md](references/delete-task.md) | "Delete task X" (irreversible — prefer finish) |
| [get-task-description.md](references/get-task-description.md) | "Show description of task X" |
| [set-task-description.md](references/set-task-description.md) | "Set / update description on task X" (destructive-overwrite warning inside) |
| [set-task-estimate.md](references/set-task-estimate.md) | "Set estimate on task X" (paid feature — HTTP 402 on free plans) |
| [delete-task-estimate.md](references/delete-task-estimate.md) | "Remove estimate from task X" |
| [set-task-reminder.md](references/set-task-reminder.md) | "Remind me about task X at …" |
| [delete-task-reminder.md](references/delete-task-reminder.md) | "Cancel reminder on task X" |
| [get-task-public-link-not-reachable.md](references/get-task-public-link-not-reachable.md) | "Share task X publicly" (UI-only currently) |
| **Subtasks** | |
| [list-subtasks.md](references/list-subtasks.md) | "List subtasks of task X" |
| [create-subtask.md](references/create-subtask.md) | "Add a subtask to task X" |
| [get-subtask-detail.md](references/get-subtask-detail.md) | "Show subtask details" |
| [finish-subtask.md](references/finish-subtask.md) | "Finish subtask X" |
| [reactivate-subtask.md](references/reactivate-subtask.md) | "Reopen subtask X" |
| [delete-subtask.md](references/delete-subtask.md) | "Delete subtask X" |
| [nested-subtasks-not-supported.md](references/nested-subtasks-not-supported.md) | User asks for sub-subtasks / three-level nesting |
| **Comments** | |
| [list-comments.md](references/list-comments.md) | "List comments on task X" |
| [create-comment.md](references/create-comment.md) | "Add a comment to task X" |
| [edit-comment.md](references/edit-comment.md) | "Edit comment X" / "attach file to existing comment" |
| [delete-comment-not-supported.md](references/delete-comment-not-supported.md) | User asks to delete a comment (workaround inside) |
| **Labels** | |
| [list-project-labels.md](references/list-project-labels.md) | "List labels in project X" |
| [create-project-label.md](references/create-project-label.md) | "Add a label to project X" |
| [edit-project-label.md](references/edit-project-label.md) | "Rename / recolor project label X" |
| [delete-project-label.md](references/delete-project-label.md) | "Remove project label X" |
| [list-task-label-colors.md](references/list-task-label-colors.md) | "What colors can a task label be?" / discover valid label colors before create / add |
| [create-task-label.md](references/create-task-label.md) | "Create a task label in the global pool" |
| [add-task-label-to-task.md](references/add-task-label-to-task.md) | "Add label X to task Y" (by UUID or name) |
| [remove-task-label-from-task.md](references/remove-task-label-from-task.md) | "Remove label X from task Y" |
| **Custom Fields** | |
| [list-custom-field-types.md](references/list-custom-field-types.md) | "What custom field types are supported" |
| [list-custom-fields-in-project.md](references/list-custom-fields-in-project.md) | "List custom fields on project X" |
| [create-custom-field.md](references/create-custom-field.md) | "Add a custom field to project X" |
| [rename-custom-field.md](references/rename-custom-field.md) | "Rename custom field X" |
| [delete-custom-field.md](references/delete-custom-field.md) | "Delete custom field X" |
| [restore-custom-field.md](references/restore-custom-field.md) | "Restore custom field X" |
| [set-custom-field-value.md](references/set-custom-field-value.md) | "Set field X on task Y to Z" (text / number / date / bool / link) |
| [delete-custom-field-value.md](references/delete-custom-field-value.md) | "Clear field X on task Y" |
| [list-enum-options.md](references/list-enum-options.md) | "What enum options does field X have" |
| [create-enum-option.md](references/create-enum-option.md) | "Add option X to enum field Y" |
| [edit-enum-option.md](references/edit-enum-option.md) | "Rename / recolor enum option X" |
| [delete-enum-option.md](references/delete-enum-option.md) | "Remove enum option X" (safe — fails if in use) |
| [force-delete-enum-option.md](references/force-delete-enum-option.md) | "Force-delete enum option X" (removes from all tasks) |
| [set-enum-value-on-task.md](references/set-enum-value-on-task.md) | "Set enum field X on task Y to option Z" |
| **Notes** | |
| [create-note.md](references/create-note.md) | "Create a note on project X" |
| [get-note.md](references/get-note.md) | "Show note X" |
| [edit-note.md](references/edit-note.md) | "Edit note X" |
| [delete-note.md](references/delete-note.md) | "Delete note X" |
| **Time Tracking** | |
| [start-timer.md](references/start-timer.md) | "Start tracking time on task X" / "start my timer" |
| [get-timer-status.md](references/get-timer-status.md) | "What am I tracking" / "is my timer running" |
| [stop-timer.md](references/stop-timer.md) | "Stop my timer" |
| [edit-timer.md](references/edit-timer.md) | "Switch the timer to task Y" / "change the timer note" |
| **Work Reports** | |
| [create-work-report.md](references/create-work-report.md) | "Log N minutes on task X" / "add a work report" |
| [edit-work-report.md](references/edit-work-report.md) | "Edit work report X" |
| [delete-work-report.md](references/delete-work-report.md) | "Delete work report X" |
| [list-work-reports.md](references/list-work-reports.md) | "List my work reports" / "reports on project X" |
| **Files & Documents** | |
| [list-project-files.md](references/list-project-files.md) | "List files in project X" |
| [upload-and-attach-file-to-comment.md](references/upload-and-attach-file-to-comment.md) | "Attach a file to a comment on task X" |
| [upload-and-attach-file-on-task-create.md](references/upload-and-attach-file-on-task-create.md) | "Create task with attached file" |
| [upload-and-attach-file-to-description.md](references/upload-and-attach-file-to-description.md) | "Attach a file to the task description" |
| [download-file.md](references/download-file.md) | "Download file X" / "file says 'resource does not exist'" |
| [detach-files-from-comment.md](references/detach-files-from-comment.md) | "Remove files from comment X but keep the comment" |
| **Templates** | |
| [list-template-projects.md](references/list-template-projects.md) | "List my templates" |
| [create-project-from-template.md](references/create-project-from-template.md) | "Create project from template X" |
| [create-tasklist-from-template.md](references/create-tasklist-from-template.md) | "Create tasklist from template X" |
| [create-task-from-template.md](references/create-task-from-template.md) | "Create task from template X" |
| **Pinned Items** | |
| [list-pinned-items.md](references/list-pinned-items.md) | "Show pinned links on project X" |
| [create-pinned-item.md](references/create-pinned-item.md) | "Pin a link to project X" |
| [delete-pinned-item.md](references/delete-pinned-item.md) | "Unpin item X" |
| **Notifications** | |
| [list-notifications.md](references/list-notifications.md) | "Show my notifications" / "what's unread" |
| [mark-notification-read.md](references/mark-notification-read.md) | "Mark notification X as read" |
| [mark-notification-unread.md](references/mark-notification-unread.md) | "Mark notification X as unread" |
| **Events** | |
| [list-events.md](references/list-events.md) | "Audit log" / "recent activity on project X" / "what changed last week" |
| **Out of Office** | |
| [check-out-of-office.md](references/check-out-of-office.md) | "Is X out of office" / "show my OOO status" |
| [enable-out-of-office.md](references/enable-out-of-office.md) | "Set OOO from D1 to D2" |
| [disable-out-of-office.md](references/disable-out-of-office.md) | "Cancel my OOO" |
| **Invoices** | |
| [list-issued-invoices.md](references/list-issued-invoices.md) | "Show invoices on project X" |
| [get-invoice-detail.md](references/get-invoice-detail.md) | "Show invoice X" |
| [mark-invoice-as-invoiced.md](references/mark-invoice-as-invoiced.md) | "Mark invoice X as invoiced" |
| [invoices-create-not-supported.md](references/invoices-create-not-supported.md) | "Create / issue / vystavit fakturu" (UI-only — API has no create endpoint) |
| **Search** | |
| [search-freelo.md](references/search-freelo.md) | "Find / search for task / project / comment / file by keyword" |

## Fallback to the OpenAPI docs

For endpoints not covered by the references above, use `WebFetch` against <https://api.freelo.io/docs/v1/freelo-api>. The references cover the common 95% of Freelo operations.
