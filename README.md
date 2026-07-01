# Motley MCP Server

Live NYC family intelligence for AI assistants.

Motley exposes the same sourced NYC family-data behind https://motley.nyc as MCP tools. Assistants can query structured data for NYC neighborhoods, schools, admissions, safety, health, family resources, kids' activities, and neighborhood comparisons.

## Hosted endpoint

```text
https://mcp.motley.nyc/mcp
```

Transport: Streamable HTTP
Auth: none required
Website/docs: https://motley.nyc/mcp

## Claude Desktop / stdio bridge

For MCP clients that only support stdio, use `mcp-remote`:

```json
{
  "mcpServers": {
    "motley": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://mcp.motley.nyc/mcp"
      ]
    }
  }
}
```

## Catalog metadata

This repository is intended to be the public metadata home for listing Motley in MCP catalogs and registries.

Official MCP Registry server name:

```text
io.github.HypnoLabs-io/motley-mcp
```

Remote registry record lives in `server.json`.

Smithery/static server card candidate lives in `.well-known/mcp/server-card.json` and can also be deployed to:

```text
https://motley.nyc/.well-known/mcp/server-card.json
```

## Tools

Motley currently exposes 14 tools:

- `search-neighborhoods` — Search and filter NYC neighborhoods by borough or name. Returns NTA codes, names, boroughs, and centroid coordinates.
- `get-neighborhood` — Get full details for a single NYC neighborhood including demographics, raw 0-100 scores, location, and a family-readable interpretation (labeled score bands + cached editorial blurbs). Use the nta_code from search-neighborhoods.
- `get-neighborhood-summary` — Get a decision-ready, family-readable summary of one NYC neighborhood: a one-paragraph fit verdict, labeled score bands (e.g. 'Peace of Mind: strong (78/100)'), and cached editorial blurbs — with raw scores still available. The interpretation-first front door; start here when a family wants an answer, not a data dump.
- `get-schools` — Get schools within a neighborhood — public, charter, and private. Includes quality ratings, grade levels, contact info. Use count_only for a quick summary, or limit/offset to page the full list (the response reports total/returned).
- `get-resources` — Get family resource counts for a neighborhood: recreation centers, playgrounds, libraries, hospitals, farmers markets, ADA subway stations.
- `get-activities` — Get kids' activities and businesses in a neighborhood — classes, sports, arts, tutoring, etc. from Foursquare data. Use count_only for category breakdown, or limit/offset to page the full list (reports total/returned). Each place's last_refreshed is when we last verified it with Foursquare.
- `get-safety` — Get safety and health environment indicators for a neighborhood: crime complaints, collisions, HPD violations, air quality, lead levels, asthma rates — plus a family-readable interpretation of the Peace of Mind and Air & Water Quality scores.
- `get-comparable` — Find neighborhoods most similar to a given one based on a 9-dimension score vector (family density, wealth, education, safety, transit, etc.). Returns similarity scores.
- `find-neighborhood` — Find which NYC neighborhood (NTA) an address or coordinate is in. Pass a NYC address (geocoded via OpenStreetMap) OR lat+lng. Returns the NTA code, name, borough, and a decision-ready family-fit summary. Use this to turn a real-world location into a neighborhood you can pass to the other tools.
- `rank-neighborhoods` — Rank NYC neighborhoods by what a family cares about. Pass per-metric weights (family density, schools, affordability, neighborhood character, transit, safety, air & water quality) and/or ranked priority keywords; optionally scope to one borough. Returns the top neighborhoods with a fit score and the metric breakdown — a deterministic weighted sort (the per-child Smart Match stays in the Motley app).
- `compare-neighborhoods` — Compare 2–6 NYC neighborhoods side by side across the 9 family-fit metrics (Family Fit, Families, Affordability, Schools, Neighborhood Character, Transit, Safety, Health, Activities). Pass an array of NTA codes; returns a per-metric matrix with each neighborhood's band and the per-metric leader.
- `get-admissions-methods` — Get the full NYC school-admissions reference: every entry point from 3-K and Pre-K through Kindergarten, Gifted & Talented, and the 8 middle/high admission methods (SHSAT, audition, screened, Ed-Opt, limited unscreened, unscreened, zoned, charter lottery) — each with its typical application window. The methods carry a live program count; optional band/governance filters narrow that count. Knowledge only.
- `explain-admissions-method` — Explain one NYC admission entry point or method in full — the plain-English explainer plus its typical deadline window. Pass a method key (shsat, audition, screened, ed_opt, limited_unscreened, unscreened, zoned, charter_lottery) or an early entry point (three-k, pre-k, kindergarten, gifted-talented).
- `get-admissions-demand` — Get how competitive admission to one specific school is, program by program: seats vs. applicants, applicants-per-seat, offer/fill rate, and a family-readable competitiveness tier (Accessible / Competitive / Highly Competitive). Pass a school DBN (from get-schools). Sourced from MySchools — present mostly for screened/choice high-school programs; zoned, charter-lottery, and private schools usually have none.

## Ownership

Maintained by Motley / HypnoLabs.
