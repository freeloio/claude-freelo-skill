# Deleting a comment is not supported via API

Load when the user asks to delete a comment. Direct them to the Freelo UI, or suggest editing to empty text.

### Delete comment — **not supported via API**

The REST API has no delete endpoint for comments. `DELETE /comment/{id}`, `/comments/{id}`, `/comment/{id}/delete` — all return 404. If the user wants to remove a comment, ask them to delete it through the Freelo web UI, or edit the content to empty text as a workaround.
