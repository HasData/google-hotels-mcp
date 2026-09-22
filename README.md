# Google Hotels MCP Server

<!-- mcp-name: com.hasdata/google-hotels -->

A hosted Model Context Protocol (MCP) server that gives Claude, Cursor, Windsurf and any other MCP client one Google Hotels tool. Search hotels and vacation rentals for a destination and a pair of dates, then open any property in full with the rate every booking site is charging, ratings, a review breakdown by topic, amenities, photos and coordinates, all as structured JSON, with no Google account and no Places API quota.

**1,000 free credits every month, no card required**, which is about 100 hotel searches.

```
https://mcp.hasdata.com/mcp?apis=google_travel_hotels
```

[![Glama score](https://glama.ai/mcp/servers/HasData/google-hotels-mcp/badges/score.svg)](https://glama.ai/mcp/servers/HasData/google-hotels-mcp)
[![tool contract](https://github.com/HasData/google-hotels-mcp/actions/workflows/contract.yml/badge.svg)](https://github.com/HasData/google-hotels-mcp/actions/workflows/contract.yml)
[![MCP](https://img.shields.io/badge/MCP-remote%20%7C%20streamable%20HTTP-6366f1?style=flat-square)](https://modelcontextprotocol.io)
[![Tools](https://img.shields.io/badge/tools-1-10b981?style=flat-square)](#tools)
[![npm](https://img.shields.io/npm/v/@hasdata/google-hotels-mcp?style=flat-square&logo=npm&label=npm&color=cb3837)](https://www.npmjs.com/package/@hasdata/google-hotels-mcp)
[![PyPI](https://img.shields.io/pypi/v/hasdata-google-hotels-mcp?style=flat-square&logo=pypi&logoColor=white&label=PyPI&color=3775a9)](https://pypi.org/project/hasdata-google-hotels-mcp/)
[![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)](LICENSE)

## Contents

- [What you need](#what-you-need)
- [Quick start](#quick-start)
- [Example prompts](#example-prompts)
- [Tools](#tools)
- [Errors and failure paths](#errors-and-failure-paths)
- [Pricing, free tier and limits](#pricing-free-tier-and-limits)
- [Tool selection](#tool-selection)
- [How it compares](#how-it-compares)
- [FAQ](#faq)
- [HasData links](#hasdata-links)
- [Development](#development)
- [Contributing](#contributing)
- [License](#license)

## What you need

An MCP client and a HasData API key from the [dashboard](https://app.hasdata.com/sign-up?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp), free to create with no card, and the free tier covers about 100 calls a month at the 10-credit rate. This is a remote server, so the simplest path is a URL and an `x-api-key` header, with no container to run and no Google Cloud project anywhere in the flow. A client that only speaks stdio reaches it through a thin launcher, published as `@hasdata/google-hotels-mcp` on npm and `hasdata-google-hotels-mcp` on PyPI, shown below.

## Quick start

The server URL is the same for every client. We run it hands-on in Claude Code and Claude Desktop. The other blocks follow each client's own documented format for a remote server.

| Field | Value |
| :--- | :--- |
| URL | `https://mcp.hasdata.com/mcp?apis=google_travel_hotels` |
| Transport | HTTP, streamable |
| Auth header | `x-api-key: HASDATA_API_KEY` |

Clients with OAuth support can add the same URL as a connector and sign in without putting a key in a config file.

<details>
<summary><b>Claude Code</b></summary>

```bash
claude mcp add --transport http google-hotels "https://mcp.hasdata.com/mcp?apis=google_travel_hotels" \
  --header "x-api-key: HASDATA_API_KEY"
```

</details>

<details>
<summary><b>Claude Desktop</b></summary>

Settings, then Connectors, then Add custom connector, then paste `https://mcp.hasdata.com/mcp?apis=google_travel_hotels` and sign in.

For the config-file route, Claude Desktop loads only local (stdio) servers, so it reaches a remote server through a stdio launcher. The `@hasdata/google-hotels-mcp` package is that launcher, and it reads the key from the environment. Add this to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "google-hotels": {
      "command": "npx",
      "args": ["-y", "@hasdata/google-hotels-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

For Python instead of Node, swap the launcher for the PyPI package, which `uvx` runs without a manual install:

```json
{
  "mcpServers": {
    "google-hotels": {
      "command": "uvx",
      "args": ["hasdata-google-hotels-mcp"],
      "env": { "HASDATA_API_KEY": "YOUR_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Cursor</b></summary>

`~/.cursor/mcp.json` for every project, or `.cursor/mcp.json` for one:

```json
{
  "mcpServers": {
    "google-hotels": {
      "url": "https://mcp.hasdata.com/mcp?apis=google_travel_hotels",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>Windsurf</b></summary>

`~/.codeium/windsurf/mcp_config.json`. Windsurf calls the field `serverUrl`, not `url`:

```json
{
  "mcpServers": {
    "google-hotels": {
      "serverUrl": "https://mcp.hasdata.com/mcp?apis=google_travel_hotels",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

<details>
<summary><b>VS Code</b></summary>

`.vscode/mcp.json` in the workspace:

```json
{
  "servers": {
    "google-hotels": {
      "type": "http",
      "url": "https://mcp.hasdata.com/mcp?apis=google_travel_hotels",
      "headers": { "x-api-key": "HASDATA_API_KEY" }
    }
  }
}
```

</details>

## Example prompts

Prompts, not code. Paste one in and the agent picks the tool itself. Each is annotated with the calls it takes, because every successful call costs 10 credits.

> Find hotels in Barcelona for the nights of 12 to 15 November, two adults, and list the five cheapest with nightly rate, rating and class.

*One call, 10 credits. Rates, ratings and amenities all come back together.*

> Same stay, four stars and up, free cancellation only, sorted by rating.

*One call, 10 credits. Class, cancellation and sort order are filters on the one request.*

> Take the top result and show what each booking site charges for it, and whether that is a good price.

*Two calls, 20 credits. The search hands back a `propertyToken` per property, and passing it in opens that property with every source Google compares, a typical price range and Google's own verdict on the deal.*

> Vacation rentals in Lisbon that week with at least two bedrooms and a pool.

*One call, 10 credits, with `vacationRentals` on, `bedrooms` at 2 and the pool in `amenity__`.*

Name the destination the way you would say it out loud. A bare `Barcelona` is read against wherever Google believes the search is running from, so a server in California answers with Californian hotels. Either write `hotels in Barcelona` or set `gl` to `es`, and the city is pinned.

## Tools

| Tool | What it returns |
| --- | --- |
| `hasdata_google_travel_hotels_getGoogleHotels` | Properties with nightly and total rate, class, rating, review counts and a breakdown by topic, amenities, images, coordinates and nearby transit, alongside pagination and the brand tree. Given a `propertyToken` it returns that one property with the rate at every source Google compares, address, phone and a typical price range. 10 credits a call |

One tool, read-only, covering both halves of the site. Without a `propertyToken` it searches. With one it opens a single property.

The samples below are trimmed from real calls, and hotel rates move daily. Read them as a shape.

A sample is the payload, not the whole response. A `tools/call` result carries one text block, and that text is itself JSON holding `url`, `status`, `text` and `json`, with the scraped data under `json`. From a raw JSON-RPC response the path is `result.content[0].text`, parsed, then `.json`. A chat client unwraps that for you and code talking to the endpoint directly does not.

### Search hotels and vacation rentals

[`hasdata_google_travel_hotels_getGoogleHotels`](https://docs.hasdata.com/apis/google-travel/hotels?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp)

Properties for a destination and a stay, with rates, ratings, amenities and images.

| Parameter | Type | Required | Notes |
| :--- | :--- | :--- | :--- |
| `q` | string | yes | Destination, neighbourhood or hotel name. Phrase it as `hotels in Barcelona` rather than `Barcelona`, or pin the country with `gl` |
| `checkInDate` | string | yes | `YYYY-MM-DD` |
| `checkOutDate` | string | yes | `YYYY-MM-DD` |
| `adults` / `children` | number | | Guest mix, 1 to 6 adults and up to 5 children, 6 guests in total |
| `childrenAges` | string | | Comma-separated ages, one per child, such as `5,8` |
| `sortBy` | string | | `lowestPrice`, `highestRating` or `mostReviewed`. Google's own order when absent |
| `minPrice` / `maxPrice` | number | | Per night, in the selected currency |
| `rating` | string | | `threePointFivePlus`, `fourPlus` or `fourPointFivePlus` |
| `hotelClass` | string | | Star classes to keep, comma-separated, such as `4,5` |
| `propertyType__` | array | | Property types such as `hotelResort`, prefixed `hotel*` for hotels and `vacation*` for rentals |
| `amenity__` | array | | Amenities such as `hotelFreeWifi` or `hotelPool`, same prefix rule |
| `brands` | string | | Brand ids to keep. The search response carries the whole brand tree with its ids |
| `freeCancellation` / `specialOffers` / `ecoCertified` | boolean | | Narrow to properties carrying that flag |
| `vacationRentals` | boolean | | Search rentals instead of hotels |
| `bedrooms` / `bathrooms` | number | | Minimums, rentals only |
| `currency` / `gl` / `hl` | string | | Currency, and the country and language of the search |
| `nextPageToken` | string | | Next page, taken from `pagination` |
| `propertyToken` | string | | Switches the call from a search to one property in full |

Results arrive under `properties`, roughly twenty per page. Each property carries `name`, `type`, `description`, `link`, `gpsCoordinates`, `ratePerNight` and `totalRate` (each with the raw string and an `extracted*` number), `hotelClass` with `extractedHotelClass`, `overallRating`, `reviews`, `locationRating`, a `ratings` histogram, `reviewsBreakdown` with positive and negative mentions per topic, `amenities`, `images` and `nearbyPlaces` with walking and transit times. `searchInformation.totalResults` reports how many Google has, `pagination` carries `nextPageToken`, and `brands` lists the brand tree with the ids the `brands` filter takes.

> Rates are quoted two ways. `lowest` includes taxes and fees, `beforeTaxesFees` does not, and a property may carry only one of them. Compare like with like before ranking.

```json
{
  "name": "Casa Gràcia",
  "propertyToken": "ChcI0r6-2uyF7rIuGgsvZy8xdmo2bnNxZhAB",
  "type": "hotel",
  "description": "Cozy quarters in a hip lodging with dining & a lively bar, plus a kitchen, a library & free Wi-Fi.",
  "link": "https://room00hostel.com/barcelona/casa-gracia-hostel/",
  "gpsCoordinates": { "latitude": 41.3974523, "longitude": 2.1593339 },
  "ratePerNight": { "beforeTaxesFees": "US$61", "extractedBeforeTaxesFees": 61 },
  "totalRate": { "beforeTaxesFees": "US$182", "extractedBeforeTaxesFees": 182 },
  "hotelClass": "4-star hotel",
  "extractedHotelClass": 4,
  "overallRating": 3.9,
  "reviews": 3811,
  "locationRating": "4.6",
  "amenities": ["Breakfast", "Air conditioning", "Airport shuttle", "Kid-friendly"],
  "nearbyPlaces": [
    { "name": "La Pedrera - Casa Milà", "transportations": [{ "type": "Walking", "duration": "5 min" }] }
  ]
}
```

### One property in full

Pass `propertyToken` from any search result back into the same tool, keeping the dates, and the answer is that property alone. On top of the search fields it adds `prices`, the rate at every source Google compares, `featuredPrices` for the sponsored ones, `typicalPriceRange` for the stay, `deal` and `dealDescription` when Google marks the price as a good one, `address`, `phone`, `directions`, `amenitiesDetailed` grouped by category, `excludedAmenities` and `otherReviews` from Tripadvisor and the rest.

```json
{
  "prices": [
    {
      "source": "Booking.com",
      "numGuests": 2,
      "ratePerNight": { "lowest": "$61", "beforeTaxesFees": "$39", "extractedLowest": 61 }
    }
  ],
  "typicalPriceRange": { "lowest": "$51", "highest": "$68", "extractedLowest": 51, "extractedHighest": 68 },
  "deal": "34% less than usual",
  "dealDescription": "Great Deal",
  "address": "Pg. de Gràcia, 116Bis, Gràcia, 08008 Barcelona, Spain",
  "phone": "+34 931 74 05 28"
}
```

## Errors and failure paths

Your client almost never sees an HTTP error code from a tool call. The MCP layer answers 200 and puts the failure inside the result, with `isError` set to `true` and the reason as text. The agent reads a message where you might expect a status line.

**A wrong key surfaces as tool output, not as a failed connection.** `tools/list` accepts any non-empty key and returns the tool, so the client completes its handshake and shows green. The first tool call then comes back with `isError: true` and the text `HasData API error: 401 Unauthorized`. Watch for that string, because nothing earlier in the flow reports the problem.

**A missing key is the one real HTTP error.** Authorization runs before any tool, and the connection itself fails with 401. CORS headers are present, and a browser client reads the status and not an opaque network failure.

**A missing date is rejected before it becomes a search.** Drop `checkOutDate` and the call fails validation with 422, naming the field.

**The dangerous failure is silent and geographic.** A destination Google cannot place, and a bare city name read from the wrong country, both return 200 with a full page of properties somewhere else entirely. We searched `Qwertyville` and got eighteen real hotels near the datacentre the request left from. Check `gpsCoordinates` on the first result, or pass `gl`, before trusting a list.

**Dates that make no sense are quietly repaired rather than refused.** A check-out before the check-in, or a stay in 2020, still answers 200 with properties, because Google normalises the range instead of erroring. Validate the dates on your side if they come from a model.

Results that carry data also carry a `requestMetadata.id` worth quoting in support.

## Pricing, free tier and limits

Every Google Hotels call costs **10 credits per successful call**. Response size does not change the price, and opening one property costs the same as a search.

The free tier is **1,000 credits every month with no card**, which is about 100 searches. It renews with the billing cycle, so a low-volume agent runs on the free tier indefinitely.

Paid plans start at **$59 a month** for 200,000 credits, which is 20,000 searches, or **$2.95 per 1,000 searches**. The unit price falls with volume to **$0.83 per 1,000** on the largest plan, and annual billing takes ten months of the monthly rate for twelve. Current figures live on the [pricing page](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp).

Your plan also sets concurrency: 1 request at a time on the free tier, 5 on Startup, 15 on Basic, and 50 to 500 across the Growth tiers. Handle the overflow case defensively in anything unattended.

A request that comes back non-200 is not billed. Opening a property after a search is a second call, so budget for it.

## Tool selection

The `apis` query parameter decides which tools your agent sees. Fewer tools means less context spent on tool definitions, and fewer chances for the model to reach for the wrong one.

```
?apis=google_travel_hotels           the one tool in this repo
?apis=google_travel                  add Google Flights
?apis=google_travel_hotels,airbnb    hotels plus Airbnb stays
?apis=google_travel_hotels,booking   hotels plus Booking.com
```

The parameter takes provider names like `google_travel` and individual API names like `google_travel_hotels`. Misspelled names are ignored. If every name is wrong the request fails with 400, and the body lists both what it did not recognise and every valid value. Drop the parameter and the same endpoint exposes every HasData tool.

## How it compares

Google has never opened a public Hotels API. Hotel Center is for property owners feeding rates in, and the Places API returns a business record with photos and reviews but no rate for a date range. The alternatives are affiliate APIs from individual booking sites, each covering its own inventory and each gated behind approval.

| | Google Places API | This server |
| :--- | :--- | :--- |
| Rates for a stay | Not offered | Nightly and total, per property |
| What other sites charge | Not offered | `prices` with the source and its rate |
| Reviews | Five review snippets | Counts, a star histogram and a breakdown by topic |
| Vacation rentals | Not covered | The same tool with one flag |
| Setup | Google Cloud project, billing, quota | One key and one URL |
| Cost | Per request, after the free cap | Paid past the free tier, 10 credits a call |

**What this server does not do.** No booking and no payment. It reads rates, availability for the dates you ask about and the links Google itself points at, and hands the booking step back to you.

## FAQ

### Is there an official Google Hotels API?

No. Google runs Hotel Center for hoteliers publishing their own rates, and the Places API for business records, neither of which returns what a stay costs. This server reads the public results and returns them as structured JSON.

### What is a Google Hotels MCP server?

A server that exposes Google Hotels as a tool an AI client can call. The client sends a tool call over the Model Context Protocol, the server fetches the properties and returns structured JSON, and the model works with the result. This one exposes a single tool and runs remotely.

### Why did my search come back with hotels in another country?

Because a bare place name is resolved against Google's own idea of where the search is running, which is the datacentre the request leaves from. Write `hotels in Barcelona` instead of `Barcelona`, or set `gl` to the country code, and the destination sticks.

### How do I see what each booking site charges?

Search first, then call the tool again with the `propertyToken` of the property you care about. The answer carries `prices` with one entry per source, `typicalPriceRange` for the stay and Google's `deal` verdict when the rate is unusually low.

### Can I search vacation rentals?

Yes. Turn on `vacationRentals` and the same tool searches rentals, where `bedrooms` and `bathrooms` become useful filters.

### Can I use this together with other HasData APIs?

Yes. The `apis` parameter takes a list, and `?apis=google_travel` adds Google Flights alongside hotels. [Drop the parameter](#tool-selection) and you get everything.

### Compliance and personal data

HasData accesses publicly available data only. A platform's terms may restrict automated access, and you are responsible for your own compliance.

## HasData links

| | |
| :--- | :--- |
| Product page and request builder | [Google Hotels API](https://hasdata.com/apis/google-hotels-api?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Server documentation | [MCP server docs](https://docs.hasdata.com/mcp-server?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Every HasData tool in one server | [HasData/hasdata-mcp](https://github.com/HasData/hasdata-mcp) |
| Client walkthroughs | [MCP clients and integrations](https://hasdata.com/integrations/mcp?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Everything else we scrape | [All HasData APIs](https://hasdata.com/apis/?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Plans and credit costs | [Plans and credit costs](https://hasdata.com/prices?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Keys and usage | [HasData dashboard](https://app.hasdata.com?utm_source=github&utm_medium=syndication&utm_campaign=google-hotels-mcp) |
| Node launcher on npm | [@hasdata/google-hotels-mcp](https://www.npmjs.com/package/@hasdata/google-hotels-mcp) |
| Python launcher on PyPI | [hasdata-google-hotels-mcp](https://pypi.org/project/hasdata-google-hotels-mcp/) |

## Development

This repository is configuration and documentation for a remote server. There is no build step and nothing to containerize.

The tests in `test/` assert the tool contract, the part that can break without a commit here. They check that `?apis=google_travel_hotels` returns exactly one tool, that it still declares its required parameters, that the name has not changed, and that the key in use is actually accepted. That last check calls the tool for real and costs 10 credits, which is the price of a canary that can fail for the right reason.

```bash
# macOS and Linux
HASDATA_API_KEY=your_key_here npm test

# Windows PowerShell
$env:HASDATA_API_KEY="your_key_here"; npm test
```

The same suite runs in CI on every push and once a week on a schedule, because the upstream tool list can change without anyone touching this repository. A failure means the tool list moved, the key stopped working, or the endpoint was unreachable, and the assertion message says which.

## Contributing

Corrections to the parameter table and the response sample are the most useful contribution, because those are the parts that drift. Include the call you made and the response you got. Pull requests from forks run the suite without a key, and the live checks skip instead of going red.

## License

MIT. See [LICENSE](LICENSE).
