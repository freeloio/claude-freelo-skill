# Mark an invoice as invoiced

Load when the user asks to mark an issued invoice as invoiced and link it to an external accounting record.

```bash
# Mark as invoiced
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"url":"https://accounting.com/inv-123","subject":"Invoice #123"}' \
  "$FREELO_BASE_URL/issued-invoice/{invoice_id}/mark-as-invoiced"
```
