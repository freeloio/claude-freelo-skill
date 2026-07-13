# Edit a comment

Load when the user asks to change a comment's text, or to retroactively attach files to an existing comment.

### Edit comment

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated text"}' \
  "$FREELO_BASE_URL/comment/{comment_id}"
```

**Attach new files to an existing comment** — the `files` field works on edit too:

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Updated with file","files":[{"uuid":"abc-1234-..."}]}' \
  "$FREELO_BASE_URL/comment/{comment_id}"
```

Existing files on the comment are preserved; the new UUIDs are added. Use this to retro-attach a file to a comment you already posted.
