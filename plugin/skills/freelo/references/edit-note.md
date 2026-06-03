# Edit a note

Load when the user asks to rename or change the content of a note. HTML headings are stripped.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"New title","content":"<p>New content</p>"}' \
  "$FREELO_BASE_URL/note/{note_id}"
```

> **HTML sanitization on notes is stricter than on comments.** Notes strip heading tags (`<h1>` through `<h6>`) before storage — the text content is kept, but the wrapping tags are removed. Safe tags like `<p>`, `<ul>`, `<li>`, `<strong>`, `<em>` pass through. The entire content is also wrapped in an outer `<div>`. Example: input `<h3>Title</h3><p>body</p>` becomes `<div>Title<p>body</p></div>`. If you need structural hierarchy, use `<strong>` or `<p>` with bold rather than heading tags.
