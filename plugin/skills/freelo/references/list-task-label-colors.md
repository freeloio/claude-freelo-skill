# List accepted task-label colors

Load when the user asks what colors a task label can be, or before creating / coloring a task label and you need a valid `color` hex.

The `color` field on task labels (`POST /task-labels`, `/task-labels/add-to-task/{tid}`) accepts only a fixed palette of hex values — any other value returns `400`. This endpoint returns the current palette, so you never have to guess or hardcode it.

### Get the palette

```bash
curl -s -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  "$FREELO_BASE_URL/task-label-colors"
```

Response shape: `{"colors": [...]}` — a dict with a `colors` key (not a bare array). Each entry is `{color, display_name, is_default}`:

- **`color`** — the hex value; send this back verbatim as the label's `color`.
- **`display_name`** — human-readable name (e.g. `grey`, `aqua`); **display-only, NOT accepted as input**.
- **`is_default`** — `true` for the fallback color applied when a label is created without one (`#77787a` grey).

10 colors are returned — this is the current recommended palette (what the Freelo UI offers).

> The write endpoints still accept ~16 additional legacy hexes for backward compatibility (the full accepted set is listed in [curl-conventions.md](curl-conventions.md) → Colors), but prefer the palette this endpoint returns for new labels.
