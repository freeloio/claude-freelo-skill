# Create a custom field on a project

Load when the user asks to add a new custom field to a project. Body key is type, NOT type_uuid.

```bash
curl -s -X POST -u "$FREELO_EMAIL:$FREELO_API_KEY" -H "User-Agent: Freelo-Claude-Skill/1.1.0" \
  -H "Content-Type: application/json" \
  -d '{"name":"Story Points","type":"<type-uuid-from-get-types>"}' \
  "$FREELO_BASE_URL/custom-field/create/{project_id}"
```

> **Body key is `type`, not `type_uuid`.** Sending `type_uuid` returns `400: Parameter type not found.`

The `type` value is a UUID from `GET /custom-field/get-types`.
