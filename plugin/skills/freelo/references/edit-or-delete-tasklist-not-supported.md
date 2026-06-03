# Editing or deleting a tasklist is not supported

Load when the user asks to rename, change budget on, or delete a tasklist — these are not exposed via the REST API.

### Edit tasklist — **not supported via API**

`POST /tasklist/{id}` returns 404. Tasklists are effectively immutable after creation — you cannot rename them, change their budget, or edit any other field through the public API. Changes must be made in the Freelo UI.

### Delete tasklist — **not supported via API**

There is no `DELETE /tasklist/{id}` endpoint (returns 404). If the user wants to remove a tasklist, ask them to delete it in the Freelo UI. As a workaround, you can move tasks out of it and leave it empty.
