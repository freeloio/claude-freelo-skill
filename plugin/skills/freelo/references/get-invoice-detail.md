# Get invoice detail

Load when the user asks for the details of a specific invoice.

```bash
# Invoice detail
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  "$FREELO_BASE_URL/issued-invoice/{invoice_id}"
```
