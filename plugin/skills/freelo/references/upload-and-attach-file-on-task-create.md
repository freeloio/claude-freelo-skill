# Attach a file when creating a task

Load when the user asks to create a task with a file already attached to its initial comment.

### Upload (returns UUID claim token)

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
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

### Attach on task create

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Task","worker":12345,"comment":{"content":"With file","files":[{"uuid":"abc-1234-..."}]}}' \
  "$FREELO_BASE_URL/project/{pid}/tasklist/{tlid}/tasks"
```

#### Notes — **file attachment not supported via API**

Adding `"files":[{"uuid":"..."}]` to a `POST /project/{pid}/note` or to `POST /note/{id}` (edit) returns HTTP 200 but the `files` array on the note stays empty — the UUIDs are silently dropped. Notes do not currently accept file attachments through the public API. If the user needs files on a note, attach them to a task instead, or ask the user to upload via the Freelo web UI.
