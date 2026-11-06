# down.guide

**A free, real-time feed of outages and incidents for agents.** [down.guide](https://down.guide) reads the official status pages of about 400 cloud, SaaS, AI and developer providers every minute, normalizes their incidents into one shape, and adds evidence of its own, always labeled as ours, to raise early warnings before a provider posts. It publishes all of it as a website, RSS, Atom and JSON feeds, a REST API and an MCP server. No account, no API key.

This repository is down.guide's home on GitHub: what it is, how the data is collected, how to use it, and where to report problems. The service is built and run by [Sazabi](https://sazabi.com?utm_source=down-guide&utm_medium=github) as a free Sazabi Labs project, like [cve.now](https://cve.now); its source code is not public, and it uses no Sazabi customer data.

## How the data is collected

- **Provider status pages.** Every minute down.guide reads each provider's own public status page (Atlassian Statuspage, incident.io, Instatus, Better Stack and status.io pages, the AWS, Google Cloud, Slack and Salesforce feeds, with RSS as a fallback). Provider words are quoted verbatim and linked to the source. A page that can't be read for 15 minutes is shown as "not reporting", never as down.
- **History.** Where a status page exposes its archive, past incidents back to 2025-01-01 are imported, so uptime and rankings start before down.guide did. Imported history never becomes a feed item or a post.
- **Probes** (our evidence). One unauthenticated GET per public provider endpoint per region per minute, from Washington, San Francisco, Frankfurt and Singapore, with the User-Agent `down.guide-probe/0.1 (+https://down.guide/about#probes)`. No credentials, no bodies read; any answer below HTTP 500 counts as reachable. One region failing alone is treated as a network blip.
- **Visitor reports** (our evidence). "I'm seeing a problem" adds one to an hourly count for that provider. Reports are shown only in aggregate, deduplicated per visitor every 10 minutes, and weighed against the provider's normal rate. We keep a daily-salted hash, not your IP.
- **Social chatter** (our evidence). Counts of posts on Hacker News, Bluesky and Mastodon. Post text is never stored.

**Levels.** An *early warning* means at least two kinds of our evidence within 10 minutes before the provider has posted anything; it's worded "signs of a problem", never "outage". *Confirmed* means the provider posted; the incident keeps our first-seen time, which is the lead time. *Stayed green* means we saw signs of a problem for 15 minutes or more and the provider's page stayed operational; it's shown with the evidence and never deleted, and a provider's denial is shown next to it.

Uptime, downtime, the Down Index (a 0-100 score per confirmed incident) and the monthly reliability rankings all come from these records; the exact definitions are on [About](https://down.guide/about) and the [methodology page](https://down.guide/leaderboard/methodology).

## Feeds

One endpoint, `https://down.guide/feed`, narrowed with filters that combine freely. RSS by default; add `format=atom` or `format=json`.

| Feed                                                                 | URL                                                            |
| -------------------------------------------------------------------- | -------------------------------------------------------------- |
| Every new incident (14 days)                                         | `https://down.guide/feed`                                      |
| Some providers                                                       | `https://down.guide/feed?provider=github,vercel`               |
| Major outages only                                                   | `https://down.guide/feed?filter=major`                         |
| One category                                                         | `https://down.guide/feed?category=ai`                          |
| Open incidents only                                                  | `https://down.guide/feed?state=active`                         |
| Combined, as JSON                                                    | `https://down.guide/feed?provider=github,vercel&filter=major&format=json` |

Filters: `provider`, `category` (`cloud`, `hosting`, `data`, `ai`, `devtools`, `observability`, `auth`, `commerce`, `messaging`, `workplace`), `impact`, `region` (e.g. `us-east-1`, `eu`), `component`, `level` (`early_warning`, `confirmed`, `not_acknowledged`), `kind` (`incident`, `maintenance`, `all`), `state` (`active`, `recent`) and `filter=major`.

Status badges: `https://down.guide/badge/<provider>.svg`.

## Check your stack

Paste a `package.json`, any lockfile, `requirements.txt`, `go.mod`, a `Gemfile`, Terraform, `docker-compose.yml`, a `.env` file (only variable names are read) or a list of domains at [down.guide/stack](https://down.guide/stack). You get the providers your project depends on, why each matched, their live state, and a feed URL for exactly that set. The input is not stored.

## MCP server

A remote Streamable HTTP MCP server at `https://mcp.down.guide/mcp` (also `https://down.guide/mcp`), no authentication. Tools: `get_provider`, `list_feed`, `get_incident`, `search`, `check_stack`, `check_dependency`, `get_leaderboard`, `send_feedback`, `report_outage` and `report_affected`.

**Claude Code**

```sh
claude mcp add --transport http down-guide https://mcp.down.guide/mcp
```

**Codex**

```sh
codex mcp add down-guide --url https://mcp.down.guide/mcp
```

**Any other client:** add a remote server with the URL `https://mcp.down.guide/mcp` and no headers.

## REST API

Every MCP tool has a REST twin that returns the same JSON. No key.

| Endpoint                                   | What it returns                                        |
| ------------------------------------------ | ------------------------------------------------------ |
| `GET /api/v1/providers/<slug>`             | One provider's live state, open incidents and history  |
| `GET /api/v1/providers`                    | The catalog with live state                            |
| `GET /api/v1/status`                       | One line per provider                                  |
| `GET /api/v1/incidents`                    | The feed, with the same filters as `/feed`             |
| `GET /api/v1/incidents/<id>`               | One incident (`DWN-…`) with every provider update      |
| `GET /api/v1/search?q=`                    | Incident and provider search (`provider:`, `started:`, `has:rca`, …) |
| `POST /api/v1/check`                       | Stack check                                            |
| `GET /api/v1/dependency?name=`             | One package, host or env var to its providers          |
| `GET /api/v1/leaderboard`                  | The Down Index or the monthly reliability ranking      |

Base URL `https://down.guide`. OpenAPI: [down.guide/openapi.json](https://down.guide/openapi.json). Every page also has a markdown copy: add `.md` to the URL or send `Accept: text/markdown`. Agent docs: [llms.txt](https://down.guide/llms.txt), [llms-full.txt](https://down.guide/llms-full.txt).

## Report a problem

- **Wrong data** (a missing or duplicated incident, a wrong state, time, uptime number or summary): [open a wrong-data issue](https://github.com/sazabi/down.guide/issues/new?template=wrong-data.yml) with the down.guide URL and a link to the provider's own page.
- **A missing provider, or a package, host or env var mapped to the wrong one**: [request it](https://github.com/sazabi/down.guide/issues/new?template=missing-provider.yml) with the provider's official status page. down.guide only covers providers with a public status page.
- **A probe or early-warning false positive**, or a provider disputing a "stayed green" entry: [report it](https://github.com/sazabi/down.guide/issues/new?template=false-positive.yml). Disputes are shown next to the entry, never hidden.
- **A broken feed, page, API call or MCP tool**: [file a bug](https://github.com/sazabi/down.guide/issues/new?template=bug.yml).
- **Private feedback**: the send feedback form on any page, the `send_feedback` MCP tool, or `POST /api/v1/feedback`. A person reads every note.

A provider that is down right now isn't an issue here: check its page on down.guide, or press "I'm seeing a problem" there.

Security problems in down.guide itself go to security@sazabi.ai; see [SECURITY.md](SECURITY.md).
