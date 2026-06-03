# Get a note

Load when the user asks to read a note.

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/note/{note_id}"
```

> Note the naming inconsistency: the note's author has `author: {id, name}` (singular `name`), whereas tasks/comments use `{id, fullname}`.
