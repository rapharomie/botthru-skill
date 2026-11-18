# BotThru skill

Teach an agent to use [BotThru](https://botthru.com), a free toolbox with no signup and no API key. The same 24 tools are available over REST, [MCP](https://botthru.com/mcp) (Streamable HTTP), and A2A.

Use it to read a public page as markdown, check DNS, RDAP, time zones, dates, holidays, fiat rates, and weather, grade security headers, scan for leaked credentials, or audit a remote MCP server.

The skill is `skills/botthru/SKILL.md`. That is the layout [`npx skills add`](https://github.com/vercel-labs/skills) discovers. Install it from a project that already has Node.js 22.20 or later:

```bash
npx skills add rapharomie/botthru-skill
```

Add `-g` to install for every project, or `--skill botthru -y` to skip the prompts.

You can also call BotThru with curl and no install. Shortcuts return plain text. `Accept: application/json` returns the tool JSON.

```bash
curl -sS "https://botthru.com/r/https://example.com"
curl -sS "https://botthru.com/calc/15%25%20of%2080"
curl -sS "https://botthru.com/time/America/Sao_Paulo"
curl -sS "https://botthru.com/dns/example.com/MX"
curl -sS "https://botthru.com/fx/USD/BRL/100"
curl -sS "https://botthru.com/weather/Paris"
curl -sS "https://botthru.com/sec/https://example.com"
```

`15% of 80` returns `15% of 80 = 12`. The other commands are live: the clock, the rate, and the forecast change, and the header grade depends on the URL.

Add the MCP server:

```bash
claude mcp add --transport http botthru https://botthru.com/mcp
codex mcp add botthru --url https://botthru.com/mcp
```

Cursor:

```json
{
  "mcpServers": {
    "botthru": {
      "url": "https://botthru.com/mcp"
    }
  }
}
```

`tools/list` returns 24 tools. Catalog and connection strings: [llms.txt](https://botthru.com/llms.txt), [connect](https://botthru.com/connect), [OpenAPI](https://botthru.com/openapi.json), [agent card](https://botthru.com/.well-known/agent-card.json).

Do not send secrets or personal data. A secret scan masks matches and does not store the text. Ask text is stored, trimmed to 2000 characters. Raw IP addresses are not stored.

Questions: [hello@botthru.com](mailto:hello@botthru.com).

MIT License. Copyright BotThru.
