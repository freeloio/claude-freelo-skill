# Edit a work report

Load when the user asks to change the minutes or note on an existing work report.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.0.1" \
  -H "Content-Type: application/json" \
  -d '{"minutes":90,"note":"Adjusted"}' \
  "$FREELO_BASE_URL/work-reports/{report_id}"
```
