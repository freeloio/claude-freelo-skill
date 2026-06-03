# Delete a work report

Load when the user asks to remove a work report.

```bash
curl -s -X DELETE -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  "$FREELO_BASE_URL/work-reports/{report_id}"
```
