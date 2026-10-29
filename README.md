# Hexadata MCP — official data sources as tools for AI agents

Hexadata publishes [Apify Actors](https://apify.com/hexadatalabs) that turn official registers and public portals into clean JSON. Every Actor is also an **MCP tool**: any MCP client (Claude Desktop, Claude Code, Cursor, VS Code, Windsurf, custom agents) can call it through the [Apify MCP server](https://mcp.apify.com), with no code to deploy.

This repository holds the connection instructions, a ready-to-use `server.json` for MCP registries, and example prompts. The tools themselves run on Apify and are billed per result (no results, no cost).

## Tools

| Tool (Actor) | What it returns | Source | Price |
| --- | --- | --- | --- |
| [eu-tenders-ted-scraper](https://apify.com/hexadatalabs/eu-tenders-ted-scraper) | EU public tenders: contract notices, awards, buyer, deadline, value, CPV | TED Search API (EU Publications Office) | $5.00 / 1,000 |
| [sam-gov-opportunities-scraper](https://apify.com/hexadatalabs/sam-gov-opportunities-scraper) | US federal contract opportunities: solicitations, sources sought, awards, NAICS, set-asides | SAM.gov public daily extract (GSA) | $5.00 / 1,000 |
| [us-healthcare-providers-npi-scraper](https://apify.com/hexadatalabs/us-healthcare-providers-npi-scraper) | US clinics, hospitals, pharmacies, practices by specialty, city, ZIP | NPPES NPI Registry API (CMS) | $1.50 / 1,000 |
| [uk-companies-house-scraper](https://apify.com/hexadatalabs/uk-companies-house-scraper) | UK companies, new incorporations, SIC codes, officers (your free Companies House key) | Companies House API | $2.00 / 1,000 + $1.00 / 1,000 detail requests |
| [mexico-business-directory-denue](https://apify.com/hexadatalabs/mexico-business-directory-denue) | Mexican businesses from INEGI's DENUE: activity, size, address, GPS | INEGI DENUE API | $2.00 / 1,000 |
| [occ-mundial-jobs-scraper](https://apify.com/hexadatalabs/occ-mundial-jobs-scraper) | Mexican job listings with salary, company and work mode | OCC Mundial | $1.50 / 1,000 |
| [seccion-amarilla-business-scraper](https://apify.com/hexadatalabs/seccion-amarilla-business-scraper) | Mexican yellow-pages businesses: phone, WhatsApp, website, hours | Sección Amarilla | $5.00 / 1,000 |

All tools share the same field conventions (`id`, `title`/`name`, `url`, `country`, `publishedAt`, `scrapedAt`), an `onlyNewSinceLastRun` mode for scheduled runs, and a `maxResults` cap that bounds the cost of every call.

## Connect in one step

Add the Apify MCP server to your client with the tools you want. Use one Actor:

```text
https://mcp.apify.com?tools=hexadatalabs/eu-tenders-ted-scraper
```

or several, comma-separated:

```text
https://mcp.apify.com?tools=hexadatalabs/eu-tenders-ted-scraper,hexadatalabs/sam-gov-opportunities-scraper,hexadatalabs/uk-companies-house-scraper
```

Authentication: sign in with OAuth on first use (the client opens apify.com), or send your Apify token as `Authorization: Bearer <APIFY_TOKEN>`. Running tools needs an Apify account; a free account includes monthly credit.

### Claude Desktop, Cursor, Windsurf (JSON config)

```json
{
  "mcpServers": {
    "hexadata": {
      "url": "https://mcp.apify.com?tools=hexadatalabs/eu-tenders-ted-scraper,hexadatalabs/sam-gov-opportunities-scraper,hexadatalabs/us-healthcare-providers-npi-scraper,hexadatalabs/uk-companies-house-scraper,hexadatalabs/mexico-business-directory-denue,hexadatalabs/occ-mundial-jobs-scraper,hexadatalabs/seccion-amarilla-business-scraper",
      "headers": { "Authorization": "Bearer <APIFY_TOKEN>" }
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http hexadata "https://mcp.apify.com?tools=hexadatalabs/eu-tenders-ted-scraper,hexadatalabs/uk-companies-house-scraper" --header "Authorization: Bearer <APIFY_TOKEN>"
```

### VS Code (`.vscode/mcp.json`)

```json
{
  "servers": {
    "hexadata": {
      "type": "http",
      "url": "https://mcp.apify.com?tools=hexadatalabs/sam-gov-opportunities-scraper",
      "headers": { "Authorization": "Bearer ${input:apify-token}" }
    }
  }
}
```

The configurator at [mcp.apify.com](https://mcp.apify.com) generates the snippet for any client; the Apify CLI can also install it (`apify mcp install cursor`).

## Example prompts

- "Which contract notices for road maintenance were published on TED in Poland and Czechia this week, and which close before December 15?" → `eu-tenders-ted-scraper`
- "Find small-business set-aside solicitations for IT services in Virginia posted since Monday." → `sam-gov-opportunities-scraper`
- "List the physical-therapy clinics in ZIP codes starting with 787 with phone numbers." → `us-healthcare-providers-npi-scraper`
- "Which software companies were incorporated in Manchester in the last 7 days, and who are their directors?" → `uk-companies-house-scraper` (pass your Companies House key in `apiKey`)
- "Give me 50 dental clinics in Monterrey with coordinates." → `mexico-business-directory-denue`

Every Actor description and input field starts with "Use this when…", so agents pick the right tool and fill the input without extra prompting. Set `maxResults` in the prompt ("at most 100") to bound the cost.

## From code instead of MCP

The same Actors are available through the Apify API and client libraries:

```python
from apify_client import ApifyClient
client = ApifyClient("<APIFY_TOKEN>")
run = client.actor("hexadatalabs/eu-tenders-ted-scraper").call(run_input={"searchQueries": ["software"], "countries": ["DE"], "maxResults": 100})
items = client.dataset(run["defaultDatasetId"]).list_items().items
```

Each Actor's README has curl, Python and Node examples plus scheduling and webhook notes.

## Legal and data

Hexadata Actors use official APIs or public portals, respect their terms, and exclude personal data unless you opt in (for example `includeOfficers` on Companies House or `includeContacts` on SAM.gov). Attribution to the source is included in every item (`source`).

## Support

Open an issue on the relevant Actor's Issues tab on Apify, or here. Issues are answered within 24 hours; confirmed bugs are fixed within 48 hours.
