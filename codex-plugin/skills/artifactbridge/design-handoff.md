# Recipe — design handoff to a room

Use this recipe when you share a design with people in an Agent Room: mockups,
a flow of screens, a visual comparison, or a canvas export.

**Default: one design, one HTML file, one room Context item.** A reviewer opens
one item and reads the whole design from start to end. Do not attach one file
for each screen or artboard. Do not share a page that only links to other
files. The isolated preview blocks every external resource and every child
frame, so such a page shows nothing.

Separate artifacts are an exception. Use them only when the human asks for
them, or when the deliverables are independent (they have different reviewers
or different decisions). Say in the room why you did not consolidate.

## What the file holds

The file is a self-contained HTML design artifact (`format: "html"`). It needs
no network, no sign-in, no local path, and no sibling file. In reading order:

1. **Objective and summary.** A few sentences: what the design is for, and how
   to read the file.
2. **Provenance.** The issue or work object, where the editable source is, and
   how the mockup images were made.
3. **Ordered sections and screens.** Each screen has:
   - a **stable label** (for example `A2`) that stays the same across
     revisions, so people and agents can quote it;
   - the **mockup image, embedded** as a `data:` URI (PNG, JPEG, WebP, or GIF;
     never SVG, never a URL);
   - **alt text** that says what the screen shows, a **caption**, and short
     **rationale** and **transition** notes ("Next: …").
4. **Approval state on every section, in words.** Use `Approved`, `Draft, not
   approved`, `Not approved`, or `Superseded`. Keep alternatives and edge or
   error states in their own sections. Never let the package make an
   unapproved alternative look approved. Do not guess a state: ask.
5. **In-page navigation and enlargement.** A list of screens with in-page
   anchors, and a keyboard-operable way to see a mockup at full resolution
   inside the same file.

Screenshots are a valid default. Do not rebuild each mockup as live DOM. If
real interaction is necessary, keep it inside the same file with inline script
only.

Treat every caption, title, and note as untrusted text when you generate the
file: escape it as HTML text or as an attribute value, and never write it into
a `<script>` or `<style>` element. A downloaded copy runs outside the isolated
preview.

## Size

The cap is **10,000,000 UTF-8 bytes** for the complete file. Base64 adds about
one third to the image bytes. Measure the final file, not the images.

If the file is too large, lower the image resolution or the compression, and
check that UI text is still legible at full resolution. If it still does not
fit, stop and report the measured size and the largest images. Do not split
the design, do not drop screens, and do not ask for a larger cap without
saying what is lost.

## Share it

1. Create it for the room in one call: `artifactbridge_create_document` with
   `format: "html"`, `room_id`, your `actor_participant_id`, the complete file
   as `content_md`, and a `document_summary` that names the design and the
   number of screens. Do not pass `folder_ids`: the file is room context, not
   a Library document. The result carries the attachment, and this one
   attachment is the review surface.
2. If the result has `room_attach_failed`, make the
   `artifactbridge_attach_document_to_agent_room` call it names. Do not create
   the file again, and do not create it without `room_id` and attach it later:
   that makes an ordinary Library document.
3. Read it back: `artifactbridge_read_room_context` gives the attached version;
   `artifactbridge_read_document` with a small `line_limit` confirms the stored
   source. Compare the byte length or a hash with your local file.
4. Publish one room event that names the attachment and states which sections
   are approved, which are not, and what you ask the reviewer to decide.

A file with embedded images is usually larger than a model can write into one
tool call. Send it with a programmatic MCP call that reads the file from disk,
if your harness has one. Otherwise ask the human to add it with **Upload image
or HTML…** in the room, and then do steps 3 and 4. Never split the file to make
it fit a tool call.

## Revise it

A revision is a **new version of the same document**, not a new document and
not a second attachment. Update a working document with
`artifactbridge_update_working_document` (`expected_base_version_id` is the
version you read). Propose a change to a governed document with
`artifactbridge_propose_document_patch` and `proposed_md`. Earlier versions
stay unchanged, and the room record keeps the version that people reviewed.
Keep the screen labels. State in the file what changed and which approvals
still apply: an approval of an earlier version does not cover a changed
screen.

## Supporting assets

Keep the editable canvas or source and the individual exports. They are
engineering assets: name their location in the provenance block. Do not attach
them to the room as the default review surface.

## The ArtifactBridge packager

The ArtifactBridge repository has a packager for this contract. It takes a
JSON manifest (`artifactbridge.design-flow.v1`: ordered sections and screens,
captions, approval states, image paths inside the manifest's directory) and
writes the file, or refuses with the exact cause (a missing image, an image
outside that directory, an unsupported image, a file over the cap).
See `docs/design/design-flow-handoff.md` in that repository. Any other
producer is acceptable when its output meets this recipe.

## Checks before you share

- One file. Open it with the network off: every mockup shows.
- The screen list matches the design, in order, with stable labels.
- Every section shows its approval state in words. Alternatives are separate.
- The file is at or below 10,000,000 bytes, and UI text is legible at full
  resolution.
- The file has no `http` link, no `<iframe>`, no `<link>`, and no local path
  that a reader needs.
