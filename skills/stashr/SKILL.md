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
   use it and read [references/mcp.md](references/mcp.md).
2. Otherwise, if shell access is available, run `stashr whoami --json` once.
   When it succeeds, use the CLI for the rest of the session and read
   [references/cli.md](references/cli.md).
3. If neither works, explain the missing setup. Recommend hosted MCP
   (`https://stashr.me/mcp`) for an interactive AI client and the CLI
   (`@stashr/cli`) for terminal agents or scripts. See
   [references/cli.md](references/cli.md) for CLI setup.

Do not check both transports before every request. Do not ask the user to paste
an API key into chat.

## Retrieve progressively

Follow this sequence unless the user asks for a specific item:

1. Search or browse compact results, at most 10 to start.
2. Keep the result IDs, media refs, source URLs, and `nextCursor` internally.
3. Fetch full content for only the result or small shortlist needed. For a
   shortlist of 2–20, use one batch fetch instead of one fetch per id.
4. Inspect selected images only when their pixels matter to the answer.
5. Follow `nextCursor` only when the current page is insufficient.

Use search for questions about meaning or wording; use list for newest-first
browsing with structured filters. The archive is searchable too (state
`archived` or `all`).

A cursor only works with the same command and the exact same query, filters,
and state; if any of them change, restart from the first page. Search ranking
covers the top ~200 candidates, so narrow with filters rather than paging deep.

Discovery hits carry bounded text snippets and lightweight media refs. Do not
fetch full content just to see whether a bookmark has images. Fetched content
arrives as Markdown with media inlined; quote or summarize it directly. Return
useful titles and source links to the user rather than dumping transport JSON.

If a result page reports degraded retrieval, say so instead of implying the
search was exhaustive.

## Answer count and overview questions with stats

For "how many", "most-used tags", or "breakdown by platform/month" questions,
make one stats call with the relevant filters; never page bookmarks and count
them. The tag list includes per-tag usage counts.

## Search and inspect images

Use image ranking when the request describes visual content such as objects,
materials, colors, scenes, screenshots, layouts, or inspiration, even if the
surrounding post text may not mention it. Shortlist by the matched image's
caption, alt text, and dimensions, then view only the shortlist.

One bookmark can appear more than once when several images match; treat each
matched media ref as a distinct hit. Stored images are viewable, while
remote-only images and videos can remain metadata-only.

## Filter values

- Platforms: `instagram`, `reddit`, `tiktok`, `twitter`, `web`, `youtube`.
  X is `twitter`.
- Content types: `article`, `comment`, `image`, `post`, `snippet`, `video` —
  bare (`article`) or platform-scoped (`twitter:article`).
- Authors: a username — bare (`naval`) or platform-scoped (`twitter:naval`).
- Tags: exact tag names; list tags when unsure. Filter untagged bookmarks
  with the untagged filter, not an empty tag.
- Capture dates: after (inclusive) and before (exclusive), ISO dates. A month
  is `2026-06-01` to `2026-07-01`.
- Media: `image`, `video`, `any-media`, or `text-only`; `text-only` cannot be
  combined with the others.

Invalid filter values and unknown filter names return a validation error
naming the allowed values. Correct the value; do not retry the same call.

## Change the library safely

Perform mutations only when requested or clearly necessary to complete the
request, and report success only after the transport confirms it.

Saving a URL is idempotent. Reuse the user's stated tag names; list tags first
only when the vocabulary is ambiguous. Archiving is reversible and works in
bulk. Collections are saved filters: require explicit confirmation immediately
before deleting one, and note that deletion does not delete its bookmarks.

## Handle permissions and errors

- Read permission covers discovery, retrieval, stats, and account info. Write
  permission covers saving and organization. Collection deletion additionally
  needs destructive permission.
- Errors say whether retrying can help (`retryable`, `retryAfterMs`). Back off
  for the stated time on rate limits; do not retry non-retryable errors.
- If authentication or permission fails, explain the specific missing access
  and ask the user to reconnect or adjust it. Do not silently switch accounts
  or credentials.
- A `payment_required` (402) error means agent access needs a Stashr Pro plan
  or an active trial. Check the account's plan and trial state, tell the user
  to visit https://stashr.me/settings/billing, and do not retry.
