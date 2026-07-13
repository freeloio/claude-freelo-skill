# Enable out-of-office

Load when the user asks to set OOO for themselves or another user. Body must be wrapped in an out_of_office object; keys are date_from/date_to, NOT from/to.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.2.0" \
  -H "Content-Type: application/json" \
  -d '{"out_of_office":{"date_from":"2026-07-01 00:00:00","date_to":"2026-07-14 23:59:59"}}' \
  "$FREELO_BASE_URL/user/{user_id}/out-of-office"
```

**Body rules:**

- Wrap the dates in an `out_of_office` object — sending `{"from": ..., "to": ...}` or top-level `{"date_from": ..., "date_to": ...}` both return 400.
- Keys are `date_from` / `date_to`, **not** `from` / `to`.
- Input format: space-separated local Prague time (`YYYY-MM-DD HH:MM:SS`).
- **Timezone conversion quirk**: input is interpreted as Europe/Prague local time, but the `GET` response returns them converted to UTC with `Z` suffix (e.g. input `2026-07-01 00:00:00` → response `2026-06-30T22:00:00Z` in summer). Expect the shift of `+02:00` / `+01:00`.
