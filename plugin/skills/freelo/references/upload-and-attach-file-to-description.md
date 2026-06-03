# Attach a file to a task description

Load when the user asks to attach a file to a task description. WARNING: calling /description on a task that has comments but no dedicated description can destructively overwrite the first comment.

### Upload (returns UUID claim token)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
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

The UUID from upload is a **claim token**: it's not bound to any project/task until you attach it. Attach via the entity's body using a `files` array.

**Files array shape** — **array of objects, not strings**:

```json
"files": [ { "uuid": "abc-1234-..." }, { "uuid": "def-5678-..." } ]
```

> `"files": ["abc-1234-..."]` (array of plain strings) returns `400 Every items of $files must be array`. Always wrap in an object.

### Task description with file attachments

`POST /task/{id}/description` accepts a `files` array identical to comments:

```bash
# 1. Upload
FILE_UUID=$(curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -F "file=@./spec.pdf" "$FREELO_BASE_URL/file/upload" | jq -r .uuid)

# 2. Set description with file attached
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"<p>See attached spec</p>\",\"files\":[{\"uuid\":\"$FILE_UUID\"}]}" \
  "$FREELO_BASE_URL/task/{task_id}/description"
```

> **⚠️ Destructive quirk — `/description` can overwrite an existing comment!** If the task has no dedicated description yet but already has regular comments, calling `/description` may **replace the first comment's content and files with the new description payload**. Original comment data is lost.
>
> **Safe pattern**: set the description BEFORE adding regular comments, or create the task with `comment:{...}` in the create body (which marks the seed comment as description from the start). If the task may already have comments, GET the task first, inspect `comments[]` for an entry with `is_description: true`, and only call `/description` when you know what you are replacing.
