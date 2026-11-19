# BotThru skill

Teach an agent to use [BotThru](https://botthru.com), a free toolbox with no signup and no API key. The same 26 tools are available over REST, [MCP](https://botthru.com/mcp) (Streamable HTTP), and A2A.

Use it to read a public page as markdown, check DNS, RDAP, time zones, dates, holidays, fiat rates, and weather, grade a live page for security headers, SEO, and agent-readiness, estimate lab page speed, scan for leaked credentials, or audit a remote MCP server.

The skill is `skills/botthru/SKILL.md`. That is the layout [`npx skills add`](https://github.com/vercel-labs/skills) discovers. Install it from a project that already has Node.js 22.20 or later:

```bash
npx skills add rapharomie/botthru-skill
```

Add `-g` to install for every project, or `--skill botthru -y` to skip the prompts.

## Claude Code plugin

This repository is a Claude Code marketplace named `botthru-marketplace`. The `botthru` plugin bundles the remote MCP server and this skill. The server is HTTP at `https://botthru.com/mcp`, with no headers and no API key.

In Claude Code:

```
/plugin marketplace add rapharomie/botthru-skill
/plugin install botthru@botthru-marketplace
```

To add only the MCP server:

```bash
claude mcp add --transport http botthru https://botthru.com/mcp
```

Codex: `codex mcp add botthru --url https://botthru.com/mcp`

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

## Call it with curl

Shortcuts return plain text. `Accept: application/json` returns the tool JSON.

```bash
curl -sS "https://botthru.com/r/https://example.com"
curl -sS "https://botthru.com/calc/15%25%20of%2080"
curl -sS "https://botthru.com/time/America/Sao_Paulo"
curl -sS "https://botthru.com/dns/example.com/MX"
curl -sS "https://botthru.com/fx/USD/BRL/100"
curl -sS "https://botthru.com/weather/Paris"
curl -sS "https://botthru.com/sec/https://example.com"
curl -sS "https://botthru.com/audit/https://example.com"
curl -sS "https://botthru.com/speed/https://example.com"
```

`15% of 80` returns `15% of 80 = 12`. The other commands are live: the clock, the rate, the forecast, and the grades change with the URL and the moment.

## Tools

These 26 names are the live `tools/list`, in that order. The use line is the live catalog. Each example is one real call. Named arguments are that tool's input fields.

| Tool | Use when | Example |
| --- | --- | --- |
| `read_url` | a page must be read as clean text | `GET /r/https://example.com` |
| `calculate` | exact math or unit conversion | `GET /calc/15%25%20of%2080` |
| `extract_page_data` | page metadata, links, or tables | `GET /v1/tools/extract_page_data?url=https://example.com&include=meta,headings` |
| `site_discovery` | robots.txt, llms.txt, or agent files | `GET /v1/tools/site_discovery?domain_or_url=example.com` |
| `check_url` | is a URL up | `GET /v1/tools/check_url?url=https://example.com` |
| `check_mcp_server` | find an MCP endpoint | `GET /v1/tools/check_mcp_server?url=https://botthru.com/mcp` |
| `check_http_security` | grade security headers | `GET /sec/https://example.com` |
| `scan_secrets` | leaked credentials | `GET /v1/tools/scan_secrets?url=https://example.com` |
| `check_mcp_security` | is an MCP server safe | `GET /v1/tools/check_mcp_security?url=https://botthru.com/mcp` |
| `audit_page` | a site's security and agent-readiness | `GET /audit/https://example.com` |
| `page_speed` | lab page speed | `GET /speed/https://example.com` |
| `dns_lookup` | DNS or email auth | `GET /dns/example.com/MX` |
| `rdap_lookup` | domain registration | `GET /v1/tools/rdap_lookup?domain=example.com` |
| `time_convert` | a time in another zone | `GET /v1/tools/time_convert?time=2026-11-17T15:00&from_tz=America/New_York&to_tz=Europe/Paris` |
| `date_math` | dates or business days | `GET /v1/tools/date_math?date=2026-11-17&add_business_days=3` |
| `holidays` | public holidays | `GET /v1/tools/holidays?country_code=US&year=2026` |
| `exchange_rate` | fiat FX | `GET /fx/USD/BRL/100` |
| `weather` | weather | `GET /weather/Paris?days=1` |
| `embed` | embed text | `POST /v1/tools/embed` `{"texts":["hello"]}` |
| `rerank` | rerank documents | `POST /v1/tools/rerank` `{"query":"docs","documents":["a","b"]}` |
| `fetch_page` | the same clean-text read as read_url | `GET /v1/tools/fetch_page?url=https://example.com` |
| `ask` | a sentence, answered only when a tool ran | `POST /v1/ask` `{"need":"what time is it in Chicago"}` |
| `time` | the current time | `GET /v1/tools/time?timezone=America/Sao_Paulo` |
| `uuid` | a random id | `GET /api/uuid` |
| `echo` | this request's headers | `GET /api/echo` |
| `whoami` | which bot is calling | `GET /api/whoami` |

`page_speed` is a lab A–F grade for one URL. It is not Core Web Vitals field data. `audit_page` grades security headers, SEO meta, and agent-readiness.

Catalog and connection strings: [llms.txt](https://botthru.com/llms.txt), [connect](https://botthru.com/connect), [OpenAPI](https://botthru.com/openapi.json), [agent card](https://botthru.com/.well-known/agent-card.json).

Do not send secrets or personal data. A secret scan masks matches and does not store the text. Ask text is stored, trimmed to 2000 characters. Raw IP addresses are not stored. [Privacy](https://botthru.com/privacy).

Questions: [hello@botthru.com](mailto:hello@botthru.com).

MIT License. Copyright BotThru.
