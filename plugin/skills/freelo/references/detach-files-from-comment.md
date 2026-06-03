# Detach files from a comment without deleting the comment

Load when the user asks to remove file attachments from a comment but keep the comment text.

To remove all files from a comment without deleting the comment, edit it with `files:[]`:

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"...","files":[]}' \
  "$FREELO_BASE_URL/comment/{comment_id}"
```

This preserves the content but clears attachments.

> **Files live independently of attachments.** Removing a file from a comment (editing with `files:[]`) or deleting the owning task does **not** delete the underlying file — `GET /file/{uuid}` still returns the bytes to anyone authenticated who knows the UUID. There is no public API for truly deleting an uploaded file.
