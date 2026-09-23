# Stashr over the CLI

Run `stashr help [command]` when syntax is uncertain. Always pass `--json`
(or `--compact`, which implies it) so output is stable and parseable.

## Setup

- Install `@stashr/cli` (`bun add -g @stashr/cli` or `npm i -g @stashr/cli`),
  then run `stashr login`. The default grant covers reading and writing; add
  `--access read` for lookup-only use or `--access full` when collection
  deletion is required.
- Headless automation may use `STASHR_API_KEY` from secret storage. Never put
  a key in a command argument, output, source file, or log.
- `stashr whoami --json` shows the credential's scopes and token expiry.

If the CLI rejects a documented option or command as unknown (for example
`stats`, `--since`, `--fields`, `--raw`, or multiple ids passed to `get`), the
installed binary is outdated. Ask the user to update it instead of silently
dropping the option.

## Commands

| Intent | Command |
| --- | --- |
| Search text and posts | `stashr search "<query>" --compact --limit 10` |
| Search images | `stashr search "<visual query>" --rank image --compact --limit 10` |
| Browse newest first | `stashr list --compact --limit 10` |
| Full content for one or several results | `stashr get <id> [<id> …] --json` (returns a `missing` list) |
| Download one image preview | `stashr media <id> --ref <ref> --output <path> --json` |
| Counts and breakdowns | `stashr stats --json` |
| Tag vocabulary with usage counts | `stashr tags --json` |
| Save a public URL | `stashr save <url> --json` |
| Edit note, favorite, or tags | `stashr update <id> --note … / --favorite / --add-tag …` |
| Archive or restore | `stashr archive <ids...>` / `stashr restore <ids...>` |
| List collections | `stashr collections list --json` |
| Create or update a collection | `stashr collections create <name>` / `stashr collections update <id>` |
| Delete a collection | `stashr collections delete <id> --confirm` |

- Continue a page with `--cursor "<nextCursor>"` on the same command.
- Choose archived or all bookmarks with `--state archived` or `--state all`.
- Keep output small with `--fields id,title,url` on any JSON output.
- `get --raw` returns the underlying Portable Text blocks; use it only when the
  user needs the raw structure.
- `stashr list` has no collection option: read a collection by passing its
  saved filters as flags.
- View downloaded previews with the environment's image-viewing capability,
  then delete the temporary files unless the user asked to keep them.

## Filters

`search`, `list`, and `stats` share these flags; repeat a flag for several
values:

- `--platform <id>` (the CLI also accepts `x` as an alias for `twitter`)
- `--content-type <type>`
- `--author <username>`
- `--tag <name>` with `--tag-mode include_any|include_all|exclude_any|exclude_all`
- `--media image|video|any-media|text-only`
- `--untagged`, `--favorite`, `--has-note`
- `--since <date>` (inclusive) and `--until <date>` (exclusive)

## Exit codes

| Code | Meaning | What to do |
| --- | --- | --- |
| 2 | Authentication required or session expired | Ask the user to run `stashr login` interactively, or to provide `STASHR_API_KEY` for headless use |
| 3 | Invalid input, unknown flag, or invalid filter value | Correct the call from the message; an unknown documented flag means an outdated binary |
| 4 | Not found | Report it; do not retry |
| 5 | Permission denied or plan required | Explain the missing scope or plan |
| 6 | Retryable: network, rate limit, or server error | Wait `retryAfterMs` if given, then retry |
