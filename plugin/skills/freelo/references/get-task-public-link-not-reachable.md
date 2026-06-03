# Task public link — not currently reachable via API

Load when the user asks to share a task publicly or get a public link. The endpoint returns 404 on tested plans; refer the user to the Freelo UI.

### Task public link — endpoint not currently reachable

`/task/{id}/public-link` returns 404 on both `GET` and `POST` in limited test contexts. This may be a paid feature or the public path has moved. If the user asks for a share link, point them to the Freelo UI "Share" action until this endpoint is validated against your plan.
