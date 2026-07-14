# Stashr agent skills

Agent skills for working with a private [Stashr](https://stashr.me) saved-bookmark library through hosted MCP or the official CLI.

## Install

```sh
bunx skills add DeckardGer/stashr-skills
```

Or with npx:

```sh
npx skills add DeckardGer/stashr-skills
```

The `stashr` skill prefers an existing Stashr MCP connection and falls back to the CLI once per session. It uses compact search results, explicit full-content fetches, and selected image previews to keep agent context useful and bounded.

Stashr setup documentation: [MCP](https://stashr.me/docs/mcp) · [CLI](https://stashr.me/docs/cli)
