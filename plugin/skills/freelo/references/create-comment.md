# Create a comment on a task

Load when the user asks to add a comment to a task. HTML is sanitized server-side; plain text is auto-wrapped in <div>.

### Create comment on a task

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"content":"Looks good, merging now."}' \
  "$FREELO_BASE_URL/task/{task_id}/comments"
```

> **Quirk:** Plain text is automatically wrapped in `<div>...</div>` by the API. To embed real HTML, send it directly (e.g. `<p>Paragraph with <strong>bold</strong></p>`). The API stores rich HTML; plain text is a shorthand.

> **HTML sanitization (security)**: Freelo strips dangerous tags and attributes server-side. `<script>`, `<iframe>`, `onclick`, `onerror`, and similar XSS vectors are silently removed before storage. Example: sending `<p>Hi</p><script>alert(1)</script><p onclick=alert(1)>click</p>` ends up stored as `<p>Hi</p><p>click</p>`. Safe tags and attributes (p, ul, li, strong, em, a href, img src without event handlers, etc.) pass through unchanged. You can rely on user-supplied HTML being sanitized — but do not trust the reverse (assume anything round-trips exactly).

> **HTML auto-balancing**: Unclosed or mis-nested tags are auto-closed in the correct order before storage. Example: `<p>unclosed <strong>bold <em>italic` becomes `<p>unclosed <strong>bold <em>italic</em></strong></p>`. You don't need to perfectly close your HTML — the API will fix it for you. Still, prefer sending well-formed HTML where possible so the stored output matches your intent.
