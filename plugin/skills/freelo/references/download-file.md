# Download a file

Load when the user asks to download a file by UUID. Auth is required on every file endpoint. "Resource does not exist" means the file was uploaded but never attached.

Once the file has been attached to any entity, download via the UUID:

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
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
