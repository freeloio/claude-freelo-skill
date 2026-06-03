# Unpin an item

Load when the user asks to remove a pinned link from a project.

```bash
# Delete
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/pinned-item/{pinned_id}"
```
