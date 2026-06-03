# Response formatting

Load when composing any reply that mentions a Freelo entity, or when the user reports they only see URLs without names.

## Response formatting — always link entities

When your response mentions a task, subtask, project, tasklist, comment, or note — by ID, name, or any form — **render it as a Markdown link `[name](url)`** to the Freelo web UI. The user should be able to click any entity you mention AND see what it is without clicking through. This is non-negotiable: do not force the user to ask "and what's the link?" after every answer, and do not force them to click a bare URL to find out which task they're looking at.

> ⚠️ **A bare URL is not enough.** `https://app.freelo.io/task/29234425` renders as a clickable URL, but the user sees just the URL — not the task name. Always wrap it: `[Nabídka pro ABC](https://app.freelo.io/task/29234425)`. Same rule for projects, tasklists, comments, notes.

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

**❌ Also bad** — bare URL renders as the URL itself, not the name; user can't tell which task is which without clicking each one:

> Found 3 overdue tasks:
> - https://app.freelo.io/task/29234425
> - https://app.freelo.io/task/29234500
> - https://app.freelo.io/task/29234600

**❌ Also bad — tables** (this is the most common failure mode). A table column called "Úkol" / "Task" with bare URLs as cell values is unreadable:

> | # | Úkol | Termín |
> |---|------|--------|
> | 1 | https://app.freelo.io/task/29326114 | — |
> | 2 | https://app.freelo.io/task/29318644 | 17. 4. 2026 |

**✅ Good — same table, names linked**:

> | # | Úkol | Termín |
> |---|------|--------|
> | 1 | [Marketing strategie Q2](https://app.freelo.io/task/29326114) | — |
> | 2 | [Newsletter — duben](https://app.freelo.io/task/29318644) | 17. 4. 2026 |

### Rules

1. **Link text = human-readable name** of the entity, NEVER the URL. Use `[Task name](url)`, not `<url>` and not the URL on its own line. If the name is genuinely unknown after fetching the resource, fall back to `Task #{id}` / `Project #{id}` / `Comment #{id}` — but do this only when you have already tried to retrieve the name and it isn't available (e.g. response was archived/deleted). Never use the fallback as a shortcut.
2. **Every entity mention in the response gets a link** — including when listing multiple, quoting back an entity from the user's request, or confirming after a create/edit. If you mention the same task 3 times in a reply, link it every time (user reads linearly, not scanning). **This applies in every rendering context: prose paragraphs, bulleted/numbered lists, table cells, headings — without exception.** If you find yourself putting a URL in a table cell, stop and turn it into `[name](url)` first.
3. **For subtasks** → use the `task_id` field from the API response (NOT `id`). The subtask-specific `id` is not routable in the web UI.
4. **For comments** → you need both `task_id` (the comment's parent task) and `comment_id` (from the create/edit response, or from the task's `comments[]` array).
5. **Do not link entities that don't have an ID yet** (e.g. a task you're about to create). Link it AFTER the create call returns with the ID.
6. **Projects** — the `/tasklists?layout=kanban` suffix is the canonical "main project view". If the user is looking for a specific sub-view (e.g. Calendar, Reports), you can drop the suffix and link just `/project/{id}/` — but default to the kanban URL.
7. **Do not link hypothetical entities** — only link what exists in the API data you fetched.

### Fallback when the renderer can't show link text

Some Markdown renderers — most notably Claude Code's **terminal UI** inside narrow contexts (table cells, inline lists) — collapse `[name](url)` to just the URL. The link text vanishes and the user sees only `https://app.freelo.io/...`, defeating the whole point.

**Detect this and switch formats**:

- **Proactive signal** — if the user explicitly says they're in a terminal / CLI / TUI ("running in terminal", "v terminálu", "v CLI", "Claude Code CLI"), use the fallback format from the start.
- **Reactive signal** — if the user asks "why don't I see task names?", "vidím jen URL", "where are the names?", or sends back a screenshot / quote of your reply showing bare URLs in cells, immediately switch and apologize briefly.

**Fallback format — two-line plain text** (name on one line, URL on its own line below):

> 1. Strategický balíček: Volba segmentu + Fáze 1 schválení
>    https://app.freelo.io/task/29234425
>    Termín: 16. 4. 2026 ⚠️ po termínu — 1 subtask, 15 komentářů
>
> 2. Připravit prezentaci pro klienta XYZ
>    https://app.freelo.io/task/29318644
>    Termín: 17. 4. 2026 ⚠️ po termínu — 1 komentář

Why this works in every renderer: terminals autodetect URLs on their own line as clickable (cmd-click / ctrl-click), and the name on the line above is plain text that always shows. The two-line cost vs. inline links is acceptable trade-off.

**Once you switch in a conversation, stay in the fallback format for the rest of it** — the user's renderer doesn't change mid-session. Only return to `[name](url)` if the user asks for it explicitly.
