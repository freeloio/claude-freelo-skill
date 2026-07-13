# Upload and attach a file to a task comment

Load when the user asks to attach a file to an existing or new comment on a task. End-to-end: upload → attach → optional download.

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

### Attach via a new comment

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"See attached","files":[{"uuid":"abc-1234-..."}]}' \
  "$FREELO_BASE_URL/task/{task_id}/comments"
```

### Full upload-and-attach example (copy-paste)

```bash
# 1. Upload
UPLOAD_RESP=$(curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -F "file=@./report.pdf" "$FREELO_BASE_URL/file/upload")
FILE_UUID=$(echo "$UPLOAD_RESP" | jq -r .uuid)

# 2. Attach to comment on task 12345
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"Attached report\",\"files\":[{\"uuid\":\"$FILE_UUID\"}]}" \
  "$FREELO_BASE_URL/task/12345/comments"

# 3. Download it back (as a new requester would)
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -o ./downloaded.pdf "$FREELO_BASE_URL/file/$FILE_UUID"
```
