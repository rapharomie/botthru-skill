---
name: botthru
description: >-
  Use when you need to read a web page as markdown, check DNS/WHOIS/RDAP,
  convert time zones or currencies, do date math, get weather or holidays,
  grade a live page for security headers, SEO, and agent-readiness, scan text
  for leaked API keys, or audit an MCP server, without installing anything or
  needing an API key.
license: MIT
compatibility: HTTPS access to https://botthru.com. No API key.
metadata:
  author: BotThru
  version: "0.2.0"
  homepage: "https://botthru.com"
---

# BotThru

BotThru is a free toolbox for agents at `https://botthru.com` (version 0.2.0). No signup and no API key. The same 25 tools are available over REST, MCP (Streamable HTTP at `https://botthru.com/mcp`), and A2A (`message/send` at `https://botthru.com/a2a`). English. United States first.

Call the tool you already know. Use `POST /v1/ask` only when the request is still a sentence. A direct call does the work; an ask is stored text plus a tool route.

## When to use

Use BotThru to:

- Read a public page as markdown (`read_url`, alias `fetch_page`), or pull title, meta, Open Graph, JSON-LD, links, headings, feeds, and tables (`extract_page_data`).
- See which agent files a site publishes (`site_discovery`) or check status, final URL, TLS, and size (`check_url`).
- Evaluate arithmetic, percentages, or a unit conversion (`calculate`).
- Read the current time (`time`), convert a civil time between IANA zones (`time_convert`), add days or business days (`date_math`), or list public holidays (`holidays`).
- Look up DNS (`dns_lookup`) or registration data for a domain, IP, or ASN (`rdap_lookup`). RDAP results omit contact emails and street addresses.
- Convert fiat currencies (`exchange_rate`). No crypto.
- Get current weather and a short forecast (`weather`).
- Grade one live page A–F for security headers, SEO meta, and agent-readiness, and list the top fixes (`audit_page`).
- Grade security headers (`check_http_security`), scan text or one public script for leaked credentials (`scan_secrets`), or review a remote MCP server (`check_mcp_server`, `check_mcp_security`). Those MCP checks run `initialize` and `tools/list` and do not call the remote tools.
- Embed or rerank short text (`embed`, `rerank`).
- Mint a UUID, echo this request, or see how BotThru classifies the caller (`uuid`, `echo`, `whoami`).

Skip BotThru for private networks, URLs with embedded credentials, booking, payment, or sending messages. Unclear text to `/v1/ask` comes back `unsupported` or `routed`. `answered` means a tool ran.

## How to call

Prefer a GET shortcut. It returns plain text. Send `Accept: application/json` for the tool result. Percent signs and spaces in a path must be encoded (`%25`, `%20`).

| Shortcut | Tool |
| --- | --- |
| `GET /r/{url}` | `read_url`. `https://botthru.com/r/example.com` works with no scheme. |
| `GET /calc/{expression}` | `calculate` |
| `GET /time/{zone}` | `time`. City or IANA name. |
| `GET /dns/{name}` or `GET /dns/{name}/{type}` | `dns_lookup`. Type defaults to A. |
| `GET /fx/{FROM}/{TO}` or `GET /fx/{FROM}/{TO}/{amount}` | `exchange_rate`. Amount defaults to 1. |
| `GET /weather/{place}` | `weather`. Optional `days` (1–7, default 3) and `units` (`metric` or `imperial`). |
| `GET /sec/{url}` | `check_http_security` |
| `GET /audit/{url}` | `audit_page`. One grade for security headers, SEO meta, and agent-readiness, plus the top fixes. |

Every catalog tool also runs at `GET /v1/tools/{name}` with arguments as query parameters, or `POST /v1/tools/{name}` with a JSON body. That wrapper is `{"ok":true,"tool":"{name}","result":{...}}`. Shortcut JSON is the inner result, with no `tool` field.

A successful result includes `applied`: the arguments that ran, with defaults filled in. Aliases are normalized there. `date_math` `days=3` becomes `add_days` `3`. `holidays` `country` becomes `country_code`. Currency codes are uppercased (`usd` becomes `USD`).

`GET /api/time`, `GET /api/uuid`, `GET /api/echo`, and `GET /api/whoami` are the dedicated REST paths for those four tools. OpenAPI: `https://botthru.com/openapi.json`.

## Examples

Captured 2026-11-18 with curl. Clock, rate, weather, and neuron-budget values change on the next call.

### Read a page

`read_url` returns markdown. It uses `text/markdown` when the origin sends it, otherwise Workers AI toMarkdown with images off, then a local HTML strip. It respects robots.txt for BotThruFetcher and does not cache or store the page. `fetch_page` is the same tool.

```bash
curl -sS "https://botthru.com/r/https://example.com"
```

```text
Title:

URL Source: https://example.com/

Markdown Content:
This domain is for use in documentation examples without needing permission. This is not a service; avoid relying on it for testing and monitoring purposes.
```

Structured extract, discovery, and a status check use query parameters:

```bash
curl -sS "https://botthru.com/v1/tools/extract_page_data?url=https://example.com&include=meta,headings"
curl -sS "https://botthru.com/v1/tools/site_discovery?domain_or_url=example.com"
curl -sS "https://botthru.com/v1/tools/check_url?url=https://example.com"
```

`extract_page_data` for that URL returned `meta.title` `Example Domain`. `include` is a comma-separated list (`meta`, `opengraph`, `jsonld`, `links`, `headings`, `tables`, `feeds`); the default is everything except tables. `site_discovery` reported `origin` `https://example.com` and `found: false` for `robots.txt`, `llms.txt`, `sitemap.xml`, and the well-known MCP, agent, OpenAPI, and security.txt paths. `check_url` returned `status` 200, `final_url` `https://example.com/`, `tls_ok` true.

### Calculate

Arithmetic, percentages, and unit conversion. No network and no eval. `expression` is at most 500 characters. `precision` is 1–34 significant digits, default 15.

```bash
curl -sS "https://botthru.com/calc/15%25%20of%2080"
```

```text
15% of 80 = 12
```

`Accept: application/json` on that URL returned `result` `"12"`, `result_number` `12`, `normalized_expression` `"(15% * 80)"`, and `applied` `{"expression":"15% of 80","precision":15}`. The same call as `GET /v1/tools/calculate?expression=2%2B2` returned result `"4"`. `100 km to mi` returned result `"62.1371192237334"` and unit `mi`.

### DNS and registration

```bash
curl -sS "https://botthru.com/dns/example.com/MX"
```

```text
Name: example.com
Type: MX
example.com 15 0 .
```

Types: `A`, `AAAA`, `CNAME`, `MX`, `TXT`, `NS`, `CAA`, `SOA`, or `email_auth` (MX, SPF, DMARC, and the DNSSEC AD flag). REST: `GET /v1/tools/dns_lookup?name=example.com&type=MX`.

```bash
curl -sS "https://botthru.com/v1/tools/rdap_lookup?domain=example.com"
```

The result was `kind` `domain`, registrar `RESERVED-Internet Assigned Numbers Authority`, nameservers `ELLIOTT.NS.CLOUDFLARE.COM` and `HERA.NS.CLOUDFLARE.COM`, registration `1995-08-14T04:00:00Z`, expiration `2027-08-13T04:00:00Z`. Pass `ip`, `asn` (for example `AS13335`), or a single `query` instead of `domain` when that is what you have.

### Time

```bash
curl -sS "https://botthru.com/time/America/Sao_Paulo"
```

```text
Timezone: America/Sao_Paulo
Local: 2026-11-18T19:59:45-03:00
UTC: 2026-11-18T22:59:45.053Z
```

JSON adds `unix_ms`, `offset`, and `abbreviation`. `GET /api/time?tz=America/Chicago` is the same clock (`tz` or `timezone`; omitted means UTC). An unknown zone is HTTP 400.

Convert a civil time with `time`, `from_tz`, and `to_tz`:

```bash
curl -sS "https://botthru.com/v1/tools/time_convert?time=2026-11-17T15:00&from_tz=America/New_York&to_tz=Europe/Paris"
```

That returned `from.local` `2026-11-17 15:00:00` and `to.local` `2026-11-17 21:00:00`, UTC `2026-11-17T20:00:00.000Z`.

### Dates and holidays

Business days are Monday through Friday. Public holidays are not skipped.

```bash
curl -sS "https://botthru.com/v1/tools/date_math?date=2026-11-17&add_business_days=3"
```

The result date was `2026-11-20`. `applied` echoes the normalized arguments and fills the other offsets with `0` (`add_business_days` `3` on this call). `days=3` is accepted and comes back as `applied.add_days` `3`. Other query fields: `add_days`, `add_hours`, `subtract_days`, `subtract_hours`, `subtract_business_days`, and `diff` (a second `YYYY-MM-DD`).

```bash
curl -sS "https://botthru.com/v1/tools/holidays?country_code=US&year=2026"
```

The result listed 17 holidays from `date.nager.at`. The first was `2026-01-01`, `New Year's Day`. Each item has `date`, `name`, `local_name`, `global`, and `types`. `year` defaults to the current UTC year. `country_code` is an ISO code or a common country name.

### Exchange rate

Frankfurter (ECB) fiat rates. Cached about an hour. Amount is converted locally.

```bash
curl -sS "https://botthru.com/fx/USD/BRL/100"
```

```text
100 USD = 581.86 BRL (rate 5.8186)
```

JSON for that call: `date` `2026-11-18`, `rate` `5.8186`, `source` `api.frankfurter.dev`. REST shape: `GET /v1/tools/exchange_rate?from=USD&to=EUR&amount=10`. Optional `date=YYYY-MM-DD`.

### Weather

Open-Meteo. Pass `place` or `latitude` and `longitude`. Keep the attribution line.

```bash
curl -sS "https://botthru.com/weather/Paris?days=1"
```

```text
Paris, FR: 12°C now, light rain; today 12-15°C, rain likely Wednesday.
Weather data by Open-Meteo.com (CC BY 4.0)
```

JSON also included `place.timezone` `Europe/Paris`, `units` `metric`, and `days` `1`. Identical requests are cached about 10 minutes. Free for non-commercial use under 10,000 calls a day.

### Security headers

One GET of the URL, plus one check that HTTP redirects to HTTPS. Read-only. No fuzzing. Grades A–F.

```bash
curl -sS "https://botthru.com/sec/https://example.com"
```

```text
Grade F because HTTP does not redirect to HTTPS, and 6 other checks failed.
- high https-redirect: HTTP does not redirect to HTTPS Fix: Redirect every HTTP request to the same host on HTTPS with 301, then send HSTS.
```

The rest of that response listed missing HSTS, CSP, `X-Content-Type-Options`, framing protection, `Referrer-Policy`, and `Permissions-Policy`. JSON fields are `grade`, `score`, `summary`, and `findings` (`id`, `severity`, `title`, `fix`). This call scored 16.

### Audit a page

Use `audit_page` when you want one A–F grade of a live page for security headers, SEO meta, and agent-readiness, plus the top fixes. `check_http_security` is headers only.

```bash
curl -sS -X POST https://botthru.com/v1/tools/audit_page \
  -H 'content-type: application/json' \
  -d '{"url":"https://example.com"}'
```

For `https://example.com` the result was `grade` `F`, `score` `0`, and `verdict` `Grade F because HTTP does not redirect to HTTPS, and 17 other checks failed.` `sections.security` was F (16), `sections.seo` F (42), and `sections.agent_readiness` D (61). `fixes` is the short list (`section`, `severity`, `title`, `fix`). `applied.url` was `https://example.com`.

`GET /audit/https://example.com` returns the same grade as plain text (verdict, section grades, fixes). `Accept: application/json` on that shortcut is the inner result, with no `tool` field.

### Scan for leaked secrets

Paste text (up to 200,000 characters) or one public URL. Send `text` or `url`, not both. Matches are redacted to the first 4 and last 2 characters. The text and the match are not stored.

```bash
curl -sS "https://botthru.com/v1/tools/scan_secrets?url=https://example.com"
```

```json
{
  "ok": true,
  "tool": "scan_secrets",
  "result": {
    "ok": true,
    "grade": "A+",
    "score": 100,
    "summary": "Grade A+ because no credential patterns matched.",
    "findings": [],
    "applied": {
      "url": "https://example.com"
    }
  }
}
```

POST the same tool to scan text: `curl -sS https://botthru.com/v1/tools/scan_secrets -H 'content-type: application/json' -d '{"text":"..."}'`. Detectors cover AWS, GitHub, OpenAI, Anthropic, Stripe, Google, Slack, a Supabase `service_role` JWT, private keys, and high-entropy secret assignments. A 20KB JSON body was accepted. Do not send a live secret to prove the scanner works.

### MCP server and MCP audit

`check_mcp_server` connects with Streamable HTTP, then `initialize`, `notifications/initialized`, and `tools/list`. It also lists prompts or resources when advertised. It never calls a tool.

`check_mcp_security` does that handshake and returns an A–F grade. It flags missing TLS, prompt injection in tool text, hidden Unicode, lookalike tool names, shell or file-write tools, destructive tools without auth, and `Access-Control-Allow-Origin: *`.

```bash
curl -sS "https://botthru.com/v1/tools/check_mcp_security?url=https://botthru.com/mcp"
```

Live result for BotThru's own server: `grade` `F`, `score` `47`, summary `Grade F because MCP CORS allows any origin, and 3 other checks failed.` The first finding was `mcp-cors-wildcard`. `https://example.com/mcp` is not an MCP server: the same call returned `ok` false, `grade` `F`, finding `Unreadable MCP response.`

`GET /v1/tools/check_mcp_server?url=https://botthru.com/mcp` returned `status` 200, `protocol_version` `2025-11-25`, and `server_info.name` `botthru`, plus the tool list.

### Embed and rerank

Both are POST, both can stop with `budget_exhausted`, and both return `estimated_neurons` and `budget_remaining`.

```bash
curl -sS https://botthru.com/v1/tools/rerank \
  -H 'content-type: application/json' \
  -d '{"query":"docs","documents":["a","b"]}'
```

```json
{
  "model": "@cf/baai/bge-reranker-base",
  "results": [
    { "index": 0, "score": 0.15730775892734528 },
    { "index": 1, "score": 0.00786574836820364 }
  ],
  "applied": { "query": "docs", "document_count": 2 }
}
```

`POST /v1/tools/embed` with `{"texts":["hello"]}` returned model `@cf/baai/bge-m3`, one vector of 1024 dimensions, and `applied.model` `@cf/baai/bge-m3`. Up to 32 strings, 8000 characters each. Optional `model`: `@cf/baai/bge-m3` or `@cf/google/embeddinggemma-300m`. Rerank accepts up to 32 documents and does not rewrite them. The rerank result above also includes `estimated_neurons` and `budget_remaining`.

### Ask, in a sentence

`status` is `answered` when a tool ran, `routed` when Workers AI only wrote a pointer, or `unsupported` when the need was logged and nothing ran. The need is stored, trimmed to 2000 characters. About 30 asks an hour. A body over 16KB returns HTTP 413 `{"error":"payload_too_large"}`.

```bash
curl -sS https://botthru.com/v1/ask \
  -H 'content-type: application/json' \
  -H 'accept: application/json' \
  -d '{"need":"what time is it in Chicago","agent":"example"}'
```

```json
{
  "status": "answered",
  "answer": "Current time in America/Chicago: 2026-11-18T17:00:47-06:00 (UTC 2026-11-18T23:00:47.289Z). BotThru only read the clock.",
  "suggestions": ["GET /api/time?tz=America%2FChicago", "GET /api/uuid"],
  "next_steps": "Use the timestamp above. Nothing was booked, sent, or changed.",
  "id": "9c59ae4c-7c04-448d-84bd-12cae7f18a3c",
  "exchange_question": "Besides money, what could you offer in exchange for this service?",
  "offer": null
}
```

`GET /v1/ask/{id}` returns that order without the need text. `POST /v1/ask/{id}/offer` with `{"offer":"..."}` stores a non-money offer.

### UUID, echo, and whoami

```bash
curl -sS https://botthru.com/api/uuid
```

```json
{ "uuid": "da06ff76-3320-4941-b83c-c2d847eccede" }
```

`GET /api/whoami` returns `bot.name`, `bot.category`, `bot.reason`, and `categories`: `ai-crawler`, `ai-user-fetch`, `search-engine`, `other-bot`, `http-library`, `browser`, and `unknown`.

`GET` or `POST /api/echo` returns method, URL, query, Cloudflare country, colo, bot classification, and headers. `Cookie`, `Authorization`, `Proxy-Authorization`, `X-Api-Key`, `X-Auth-Token`, and connecting-IP headers are replaced with `[redacted]`. A request with `X-Api-Key: example-not-a-secret` came back `x-api-key` `[redacted]` and `cf-connecting-ip` `[redacted]`.

## MCP

Streamable HTTP, JSON-RPC 2.0, no session, no SSE. Send JSON-RPC with `POST https://botthru.com/mcp`. `GET /mcp` returns a short JSON card (`transport` `streamable-http`, `sse` false, `method` `POST`). `DELETE /mcp` returns 405: `This server is stateless and has no session to delete.` `GET /mcp/sse` returns `sse_not_offered`. `notifications/initialized` returns 202 and an empty body. Protocol versions: `2025-03-26`, `2025-06-18`, `2025-11-25`. Official registry name: `com.botthru/drive-thru`. Server card: `https://botthru.com/.well-known/mcp/server-card.json`.

`tools/list` returns 25 tools, in this order: `read_url`, `calculate`, `extract_page_data`, `site_discovery`, `check_url`, `check_mcp_server`, `check_http_security`, `scan_secrets`, `check_mcp_security`, `audit_page`, `dns_lookup`, `rdap_lookup`, `time_convert`, `date_math`, `holidays`, `exchange_rate`, `weather`, `embed`, `rerank`, `fetch_page`, `ask`, `time`, `uuid`, `echo`, `whoami`.

`?tools=read_url,calculate` limits `tools/list` and `tools/call` to those names. Unknown names are ignored. Each tool also has `https://botthru.com/mcp?tools={name}` (alias `/mcp/{name}`). Prompts advertised by the server: `audit_site_for_agents` and `read_page`. Call `prompts/list` for their arguments.

Add the server:

```bash
claude mcp add --transport http botthru https://botthru.com/mcp
```

```bash
codex mcp add botthru --url https://botthru.com/mcp
```

Cursor `mcp.json`:

```json
{
  "mcpServers": {
    "botthru": {
      "url": "https://botthru.com/mcp"
    }
  }
}
```

OpenCode `opencode.json`: `{"mcp":{"botthru":{"type":"remote","url":"https://botthru.com/mcp","enabled":true}}}`.

First call:

```bash
curl -sS https://botthru.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"calculate","arguments":{"expression":"2+2"}}}'
```

The response `structuredContent.result` was `"4"` and `isError` was false. A bad tool argument is HTTP 200 with `result.isError` true. An unknown tool name is JSON-RPC `-32602`.

## A2A

Agent card: `https://botthru.com/.well-known/agent-card.json`. No streaming and no push. A clear message runs one tool and returns a completed task.

JSON-RPC `message/send` at `https://botthru.com/a2a` for "what time is it in Chicago" returned `status.state` `completed` and an artifact named `time`. HTTP+JSON `POST https://botthru.com/v1/message:send` with `{"message":{"role":"user","parts":[{"kind":"text","text":"2+2"}],"messageId":"22222222-2222-4222-8222-222222222222"}}` returned a task whose text was `(2 + 2) = 4`. The same binding is `POST /a2a/v1/message:send`.

## Limits

Per network address, about 30 asks an hour, 60 outbound tool calls an hour, and 30 embed or rerank calls an hour. Embed and rerank also stop when their daily neuron budget is spent, so `/v1/ask` keeps the rest of the Workers AI allowance. Ask bodies over 16KB are refused. `scan_secrets` text may be up to 200,000 characters.

`read_url` blocks private addresses. `extract_page_data` is cached about 10 minutes by final URL and does not store the page. Holidays are cached about 24 hours. Weather is cached about 10 minutes. Exchange rates are cached about an hour.

## Privacy

Do not send secrets, passwords, API keys, session cookies, or personal data.

BotThru logs a salted hash of the network address, the user agent, the path, the referrer hostname, the MCP method, and the ask text (trimmed to 2000 characters). Tool calls log the tool name, whether it worked, the target hostname, and latency. Raw IP addresses are not stored. Cookies are not set. Full URLs with query strings and fetched page bodies are not stored in the tool log. A secret scan stores neither the text nor the match, and a match in the response is masked.

`echo` redacts cookie, authorization, and connecting-IP headers to `[redacted]`.

Questions: hello@botthru.com. Security contact is the same address, in `https://botthru.com/.well-known/security.txt`.
