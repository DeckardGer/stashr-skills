---
name: stashr
description: Work with the user's private Stashr saved-bookmark library through the hosted Stashr MCP server or official CLI. Use when the user asks to search, browse, retrieve, summarize, or inspect images from their saves; save a URL; get library counts or stats; manage notes, tags, favorites, or archive state; organize collections; or set up and troubleshoot Stashr agent access.
---

# Stashr

Use Stashr as a private source library. Prefer compact discovery, fetch details
only for selected results, answer count questions with stats instead of paging,
and load images only when visual inspection helps.

## Choose one transport

Resolve the transport once per session, then reuse it:

1. Inspect the tools already available. If a Stashr MCP server is connected,
   use its tools without probing the CLI.
2. Otherwise, if shell access is available, run `stashr whoami --json` once.
   Use the CLI for the rest of the session when it succeeds.
3. If neither works, explain the missing setup. Recommend hosted MCP for an
   interactive AI client and the CLI for terminal agents or scripts.

If the CLI rejects a documented option or command as unknown (for example
`stats`, `--since`, `--untagged`, or `--raw`), the installed binary is
outdated. Ask the user to update it (`bun add -g @stashr/cli` or
`npm i -g @stashr/cli`) instead of silently dropping the option.

Do not check both transports before every request. Do not ask the user to paste
an API key into chat.

## Retrieve progressively

Follow this sequence unless the user asks for a specific item:

1. Search or browse compact results.
2. Keep the result IDs, media refs, source URLs, and `nextCursor` internally.
3. Fetch the full content of only the result or small shortlist needed.
4. Inspect selected images only when their pixels matter to the answer.
5. Follow `nextCursor` only when the current page is insufficient.

Start with at most 10 discovery results (`--limit 10`). Continue a page by
passing the returned cursor back: MCP `cursor: "<nextCursor>"`, CLI
`--cursor "<nextCursor>"` — always to the same command with the exact same
query, filters, and state. Search cursors are bound to those inputs; changing
any of them invalidates the cursor, so restart from the first page instead.
Search ranking covers the top ~200 candidates — narrow with filters rather
than paging deep. Return useful titles and source links to the user rather
than dumping transport JSON.

### Search text and posts

- MCP: call `search` with `rankMode: "post"`; call `fetch` for selected IDs.
- CLI: run `stashr search "<query>" --compact --json`; run
  `stashr get <bookmark-id> --json` for selected IDs.
- Use `list_bookmarks` or `stashr list --compact --json` for newest-first
  browsing and structured filters, not semantic questions.
- The archive is searchable too: pass `state: "archived"` (or `"all"`), CLI
  `--state archived`.

Fetched content arrives as Markdown (headings, lists, links, and code
preserved; media inlined with archived URLs) — quote or summarize it
directly. CLI `get --raw` returns the underlying Portable Text blocks; only
use it when the user needs the raw structure.

Discovery hits carry bounded `text` snippets plus lightweight `media` refs.
Do not fetch full content just to see whether a bookmark has images — the
refs on the hit already answer that.

If a result page includes `degraded` (or `searchDegraded`), retrieval quality
was reduced (semantic or image ranking unavailable); say so when presenting
results instead of implying the search was exhaustive.

### Answer count and overview questions with stats

For "how many", "most-used tags", "breakdown by platform/month" questions,
make one stats call — never page bookmarks and count them:

- MCP: `library_stats` with optional filters and state.
- CLI: `stashr stats --json`, with the same filters as list/search.
- Tag popularity: `list_tags` / `stashr tags --json` includes per-tag usage
  counts.

### Filter values

- Platforms: `instagram`, `reddit`, `tiktok`, `twitter`, `web`, `youtube`.
  X is `twitter`, never `x` (the CLI aliases `x` for convenience; the MCP and
  API do not).
- Content types: `article`, `comment`, `image`, `post`, `snippet`, `video` —
  bare (`article`) or platform-scoped (`twitter:article`).
- Authors: a username — bare (`naval`) or platform-scoped (`twitter:naval`).
- Tags: exact tag names; check `list_tags` / `stashr tags --json` when unsure.
  For untagged bookmarks use `untagged: true` (CLI `--untagged`).
- Capture dates: `capturedAfter` (inclusive) and `capturedBefore` (exclusive),
  ISO dates; CLI `--since` / `--until`. A month is
  `--since 2026-06-01 --until 2026-07-01`.
- Media: `image`, `video`, `any-media`, or `text-only` — `text-only` cannot
  be combined with the others.

Invalid filter values — and unknown filter names — return a validation error
naming the allowed values. Read it and correct the value; do not retry the
same call.

### Search and inspect images

Use image ranking when the request describes visual content such as objects,
materials, colors, scenes, screenshots, layouts, or inspiration, even if the
surrounding post text may not mention it.

- MCP: call `search` with `rankMode: "image"`. Shortlist using
  `matchedMedia.caption`, alt text, dimensions, and media ref. Call
  `view_media` for up to four selected refs from one bookmark.
- CLI: run
  `stashr search "<visual query>" --rank image --compact --json`. Download a
  selected preview with
  `stashr media <bookmark-id> --ref <ref> --output <path> --json`, then use the
  environment's image-viewing capability.

One bookmark can appear more than once when several images match. Treat each
`matchedMedia.ref` as a distinct ranked hit. Load previews only for the
shortlist; stored images are viewable, while remote-only images and videos can
remain metadata-only. Remove temporary CLI preview files after use unless the
user asked to keep them.

## Change the library safely

Perform mutations only when requested or clearly necessary to complete the
request. Report success only after the transport confirms it.

| Intent | MCP | CLI |
| --- | --- | --- |
| Save a public URL | `save_bookmark` | `stashr save <url> --json` |
| Edit note, favorite, tags, or archive state | `update_bookmark` | `stashr update`, `stashr archive`, or `stashr restore` |
| Archive or restore in bulk | `archive_bookmarks` (up to 200 ids) | `stashr archive <ids...>` / `stashr restore <ids...>` |
| Inspect tag vocabulary | `list_tags` | `stashr tags --json` |
| List collections | `list_collections` | `stashr collections list` |
| Read a collection's bookmarks | `list_bookmarks` with `collectionId` | `stashr list` with the collection's filters |
| Create, update, or delete collections | `manage_collection` | `stashr collections create\|update\|delete` |

Saving is idempotent. Reuse the user's stated tag names; list tags first only
when the vocabulary is ambiguous. Archive is reversible. Require explicit
confirmation immediately before deleting a collection; collection deletion
does not delete its bookmarks.

## Handle setup and permissions

- Hosted MCP URL: `https://stashr.me/mcp`. Authentication happens through the
  user's browser and Stashr consent screen.
- CLI setup: install `@stashr/cli`, then run `stashr login`. The default
  grant covers reading and writing; add `--access read` for lookup-only use
  or `--access full` when collection deletion is required. Use
  `stashr help [command]` when command syntax is uncertain.
- Headless automation may use `STASHR_API_KEY` from secret storage. Never put a
  key in a command argument, output, source file, or log.
- Read permission covers discovery, retrieval, stats, and account info. Write
  permission covers saving and organization. Collection deletion additionally
  needs destructive permission and explicit confirmation.

Errors say whether retrying can help: `retryable` plus `retryAfterMs` in
error payloads (CLI exit code 6 = retryable). Back off for the stated time on
rate limits; do not retry non-retryable errors.

If authentication or permission fails, explain the specific missing access and
ask the user to reconnect or adjust it. Do not silently switch accounts or
credentials. `stashr whoami --json` shows the credential's scopes and token
expiry, and the MCP `get_account` tool reports plan and trial state — use them
to diagnose 403s and 402s. CLI exit code 2 (`authentication_required` /
`session_expired`) means the stored session is gone: ask the user to run
`stashr login` interactively, or to provide `STASHR_API_KEY` for headless use.
A `payment_required` (402) error means agent access needs a Stashr Pro plan or
an active trial — tell the user to check https://stashr.me/settings/billing;
do not retry.
