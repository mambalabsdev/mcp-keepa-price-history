# Keepa Price History MCP Server

[![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-keepa-price-history/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-keepa-price-history) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-keepa-price-history%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-keepa-price-history&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-keepa-price-history)](https://www.npmjs.com/package/@mambalabsdev/mcp-keepa-price-history) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-keepa-price-history)](https://www.npmjs.com/package/@mambalabsdev/mcp-keepa-price-history) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-keepa-price-history)](https://github.com/mambalabsdev/mcp-keepa-price-history/blob/main/LICENSE)

An MCP server for the Mamba Labs Keepa Price History actor on Apify. Amazon price history for a list of ASINs, on your own Keepa key.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

Give a list of ASINs or Amazon product URLs and your own Keepa API key. The actor reads the official Keepa product API in batches and returns one row per product: current Amazon, new, used, and list prices, 30, 90, and 180 day averages, all time lows and highs with dates, sales rank, bought in past month, out of stock share, and decoded history for the types you pick. Each product uses your Keepa tokens; when the bucket is empty the actor waits for it to refill, up to the wait you set.

All of the work runs on Apify. This package is a thin client that starts the actor run, waits for it, and hands back the dataset rows.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config (`claude_desktop_config.json`), Cursor, or any MCP client:

```json
{
  "mcpServers": {
    "mamba-keepa-price-history": {
      "command": "npx",
      "args": [
        "-y",
        "@mambalabsdev/mcp-keepa-price-history"
      ],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Restart the client. The `get_keepa_price_history` tool appears.

## Prerequisites

- Node.js 18 or newer.
- An Apify API token from [console.apify.com/account/integrations](https://console.apify.com/account/integrations), set as `APIFY_TOKEN`. The server lists its tool without a token; a call needs one.

## Example prompts

- "Get the Keepa price history of these 10 ASINs on my Keepa key."
- "What is the lowest price ever for B00M0QVG3W on amazon.de?"
- "Compare 90 day average prices and sales ranks for these products."

## Tool

`get_keepa_price_history`. One tool. Example arguments:

```json
{
  "asins": [
    "B00M0QVG3W"
  ],
  "marketplace": "com"
}
```

## Inputs

Every buyer facing input of the actor. Omitted inputs take the actor's own defaults.

| Input | Type | Required | Description |
|---|---|---|---|
| `api_key` | string | no | Your own Keepa API access key, 64 characters, from keepa.com/#!api (needs an active Keepa API plan). It is stored encrypted by Apify, sent only to api.keepa.com, and never written to a row or a log. With no key the run writes one free row and reads nothing. |
| `asins` | array | no | ASINs (for example B00M0QVG3W) or Amazon product URLs. Repeats are read once. In Clay, put the column inside a one item list. |
| `marketplace` | string | no | Which Amazon store to read: com (default), co.uk, de, fr, co.jp, ca, it, es, in, com.mx, or com.br. A product URL's own marketplace wins over this setting. |
| `include_history` | boolean | no | yes (default) or no. With no, rows carry current prices and statistics only, and the history column is empty. Keepa charges the same tokens either way. |
| `history_types` | array | no | Which histories to decode into the history column: amazon, new, used, sales_rank, list_price, offers_new, offers_used, rating, review_count, lightning_deal, warehouse, buy_box. Default amazon, new, sales_rank. rating and review_count need include_rating; warehouse and buy_box come only when Keepa already holds offer data for the product. |
| `history_days` | number | no | Number, default 90. Limits the history column to the last N days (1 to 9999). 0 returns the full history Keepa holds. Averages and all time lows are not affected. |
| `include_rating` | boolean | no | yes or no (default). With yes, rows carry Keepa's latest rating and review count, which costs up to 1 extra Keepa token per product. |
| `max_age_hours` | number | no | Number, default 24. Keepa refreshes a product from Amazon first when its data is older than this. 0 always refreshes (1 extra Keepa token when the data is under 1 hour old). Higher values are faster. |
| `max_asins` | number | no | Number, default 100, at most 10000. ASINs past this count are not read. |
| `max_wait_seconds` | number | no | Number, default 300, at most 3000. When your Keepa token bucket is empty, the actor waits for it to refill. Waiting is earned at 20 seconds per product row delivered in this run, up to this many seconds in total, so an empty bucket at the start ends the run at once with one free row. After that the remaining ASINs get one free row that says so. |
| `max_charge_usd` | string | no | Optional. The run stops before a charge that would pass this amount. Summary rows are always returned. |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `product-row`, one product with Keepa price history | $0.006 | $0.0057 | $0.0054 | $0.0051 |
| `apify-actor-start`, per run start | $0.005 | $0.005 | $0.005 | $0.005 |

A missing or refused key, ASINs Keepa has no data for, an empty token bucket after the wait, API errors, and refused input are free. Mamba Labs pays the platform costs of the run. Mamba Labs pays the platform costs of the run, Apify proxy traffic included.

## Full actor documentation

[apify.com/mambalabs/keepa-price-history](https://apify.com/mambalabs/keepa-price-history)

## License

MIT. See [LICENSE](LICENSE).
