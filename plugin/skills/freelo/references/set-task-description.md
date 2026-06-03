# Set or update the description of a task

Load when the user asks to set or update a task's description, with or without file attachments. WARNING: can destructively overwrite an existing first comment — read the safe-pattern blockquote before calling.

```bash
# Set / update — body key is "content", NOT "description"
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"content":"<p>Full HTML content here</p>"}' \
  "$FREELO_BASE_URL/task/{task_id}/description"
```

Response includes `{id, content, date_add, files}` — the description has its own id and can have file attachments, just like a comment.

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
