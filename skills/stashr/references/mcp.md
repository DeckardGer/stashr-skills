# Stashr over MCP

The server's tool descriptions and input schemas are authoritative for
parameters and allowed values. This file maps the workflow in SKILL.md to
tool names.

| Intent | Tool |
| --- | --- |
| Search text and posts | `search` with `rankMode: "post"` |
| Search images | `search` with `rankMode: "image"` |
| Browse newest first, or read a collection | `list_bookmarks` (`collectionId` for a collection) |
| Full content for one result | `fetch` |
| Full content for a shortlist of 2–20 | `fetch_many` with `ids` (returns a `missing` list) |
| View selected images | `view_media` (up to four refs from one bookmark) |
| Counts and breakdowns | `library_stats` |
| Tag vocabulary with usage counts | `list_tags` (most-used first, paged; `query` finds tags by name) |
| Plan, trial, and quota | `get_account` |
| Save a public URL | `save_bookmark` |
| Edit note, favorite, tags, or archive state | `update_bookmark` |
| Archive or restore in bulk | `archive_bookmarks` (up to 200 ids) |
| List collections | `list_collections` |
| Create, update, or delete a collection | `manage_collection` (delete needs `confirm: true`) |

Continue a page with `cursor: "<nextCursor>"`. Filter names are camelCase:
`capturedAfter`, `capturedBefore`, `untagged: true`. Image-ranked hits carry
`matchedMedia.ref` and `matchedMedia.caption`; pass the refs to `view_media`.
