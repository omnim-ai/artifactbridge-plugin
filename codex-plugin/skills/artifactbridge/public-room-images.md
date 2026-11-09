# Public Room images

Use this when a Room's inline images are, or may become, public with its
public link.

## What an agent can do

- List: `artifactbridge_list_room_public_images` (CLI
  `artifactbridge rooms images list <room-ref> --json`). Any member who can
  read the Room. You get the Room's proven inline images with an opaque
  `review_id`, revisions, `status` (`unreviewed`, `prepared`, `approved`) and
  the active `grant`. You never get the image, a URL, a token or a public
  handle.
- Revoke one image: `artifactbridge_revoke_room_public_image` (CLI
  `artifactbridge rooms images revoke <room-ref> --input <JSON> --json`, where
  the JSON is exactly
  `{"grantId","expectedLinkRevision","expectedGrantRevision","operationKey"}`
  from list output). Room creator or owner, workspace owner or admin, or their
  agent. The link and the original image stay. A revoked image never returns;
  only a new human approval makes a new one.
- Recover: `artifactbridge_get_room_public_image_operation` (CLI
  `artifactbridge rooms images operation <room-ref> <operation-key> --json`).
  It shows only your own result for that key in that Room.

## Human handoff

Preparing and approving a public image is a person's decision. You cannot do
it, and no tool or command recovers a public handle. Tell a Room manager to
open the Room's Share dialog in the ArtifactBridge web app.

## Failures

- `stale_link` or `stale_review`: list again, then use the new revisions.
- `not_authorized`: you do not manage the Room. Hand off to a Room manager.
- Lost answer: read the operation with the same key before you do anything
  else. Never send a changed request with the same key. `not_found` means
  nothing was recorded: list again, then use a new key.

## What revokes images without you

Hiding the event, revoking or replacing the public link, making the Room
private or transferring it, a lost manager role, a suspension, a freeze and a
workspace deletion request revoke the affected images at once. A workspace
"Allow public rooms" pause only suspends them; resume does not bring back an
image that was revoked during the pause.
