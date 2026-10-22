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

## Connect

**ChatGPT:** add a custom MCP server with the URL above and **No authentication**. Step-by-step instructions are at https://motley.nyc/mcp.

**Claude and other clients with remote MCP support:** add the URL above as a remote server; no key is needed.

### Claude Desktop / stdio bridge

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

This repository is the public metadata home for listing Motley in MCP catalogs and registries.

Official MCP Registry server name:

```text
nyc.motley/mcp
```

`server.json` is the registry record. The server card, with every tool's input and output schema, lives in `.well-known/mcp/server-card.json`.

## Tools

Motley exposes 21 tools. Every one is read-only: none writes anything, and none needs an account.

- `search-neighborhoods` — Search and filter NYC neighborhoods by borough or name. Returns NTA codes, names, boroughs, and centroid coordinates.
- `get-neighborhood` — Get full details for a single NYC neighborhood including demographics, raw 0-100 scores, location, and a family-readable interpretation (labeled score bands + cached editorial blurbs). Use the nta_code from search-neighborhoods.
- `get-neighborhood-summary` — Get a decision-ready, family-readable summary of one NYC neighborhood: a one-paragraph fit verdict, what stands out and what to weigh among its Family Fit scores (`tradeoffs`), labeled score bands (e.g. 'Peace of Mind: strong (78/100)'), and cached editorial blurbs — with raw scores still available, weekday-morning transit minutes to the city's main job centers, and a link to the neighborhood's Motley page. The interpretation-first front door; start here when a family wants an answer, not a data dump. A non-residential area (park, cemetery, airport, institution) isn't scored, and the summary says so.
- `get-schools` — Get schools within a neighborhood — public, charter, and private. Includes each school's Motley ratings out of 100 (elementary and middle schools on grades 3-8 state tests, high schools on graduation and Regents results; ratings_note says how each is built), grade levels, contact info. Use count_only for a quick summary, or limit/offset to page the full list (the response reports total/returned).
- `browse-schools` — Find NYC schools citywide by what a family wants — borough, community school district (1-32), grade band (pre-K/elementary/middle/high), school type (public/charter/private), admissions method (zoned, screened, unscreened, limited unscreened, ed-opt, audition, SHSAT, charter lottery, private, universal pre-K), and program focus (arts, sports, STEM, language, academic, clubs). Use this when the family describes the KIND of school they want; use get-schools when you already have an nta_code and want everything in that neighborhood. Pass only the filters actually stated — at least one is required, and an invented one returns nothing. Program focus is tagged on public and charter schools only.
- `find-school` — Use this to turn a school's name into its DBN — the official name, a common short form ('Brooklyn Tech', 'PS 321'), or a DBN to confirm. Returns candidate schools with their district, neighborhood, admission method and Motley page; pick the one that matches and pass its dbn to get-admissions-demand. Never guess a DBN. K-12 schools only (public, charter, private).
- `get-zoned-school` — Use this when a family asks which elementary school is zoned for a specific NYC address, such as an apartment or building they name. NYC draws school zones building by building, so pass the street address exactly as they typed it — building number and street, ideally with the borough — never the device's location. Returns the zoned K-5 school, the neighborhood, and how zoned admission works; some areas, like all of District 1, have no zoned school.
- `get-resources` — Get family resource counts for a neighborhood: recreation centers, playgrounds, libraries, hospitals, farmers markets, ADA subway stations.
- `get-activities` — Get kids' activities and businesses in a neighborhood — classes, sports, arts, tutoring, etc. Use count_only for category breakdown, or limit/offset to page the full list (reports total/returned). Each place's last_refreshed is when we last confirmed it was still listed.
- `get-safety` — Get safety and health environment indicators for a neighborhood: crime complaints, collisions, HPD violations, air quality, lead levels, asthma rates — plus a family-readable interpretation of the Peace of Mind and Air & Child Health scores.
- `get-comparable` — Find neighborhoods most similar to a given one based on a 9-dimension score vector (family density, wealth, education, safety, transit, etc.). Returns similarity scores.
- `find-neighborhood` — Find which NYC neighborhood (NTA) an address or coordinate is in. Pass a NYC address the family gave (placed with NYC's official address directory; include the borough when you know it) OR lat+lng. Returns the NTA code, name, borough, a link to the neighborhood's Motley page, and a decision-ready family-fit summary. Use this to turn a real-world location into a neighborhood you can pass to the other tools.
- `rank-neighborhoods` — Rank NYC's residential neighborhoods by what a family cares about — the same ranking as Best fit on motley.nyc. Pass ranked priority words, most important first: families, schools, affordability, character, getting-around, safety, air-and-health (and synonyms such as crime, rent, transit, commute, kids), plus two factors — parks or playgrounds (the share of homes within a 10-minute walk of an NYC Parks playground) and diversity (how evenly residents spread across five census groups). Or pass per-metric weights for the seven scores. Optionally scope to one borough. Returns the top neighborhoods with a fit score and the score breakdown, which factors counted, any priority words we don't measure (relay them — they added nothing), and a best_fit_url that opens the same ranking on motley.nyc. A deterministic weighted sort over neighborhood-level data, not a per-child school match.
- `compare-neighborhoods` — Compare 2–6 NYC neighborhoods side by side across the 9 family-fit metrics (Family Fit, Families, Affordability, Schools, Neighborhood Character, Transit, Safety, Health, Activities). Pass an array of NTA codes; returns a per-metric matrix with each neighborhood's band and the per-metric leader, plus weekday-morning transit minutes from each to the city's main job centers. Non-residential areas aren't scored and are dropped.
- `explain-score` — Use this when a family asks how one of Motley's scores is computed — a neighborhood score (Family Fit, Schools, Safety / Peace of Mind, Affordability, Getting Around, Neighborhood Character, Air & Child Health, Things to Do) or the school rating — or whether the scores are biased toward wealthier areas. Returns what the score measures, its inputs and weights, how they combine, its share of Family Fit, its known limits, its sources, and how it moves with neighborhood income (Spearman's ρ with median household income). Relay the income figure plainly; don't reassure past it.
- `find-collections` — Use this when a family asks for lists of NYC schools or neighborhoods picked by a rule rather than a ranking: zoned schools that beat their district while serving high-need students; high school programs with more seats per applicant; schools where no group is a majority and families rate them highly (kind 'schools'); or affordable family neighborhoods with such a school, and family neighborhoods where no single group is a majority (kind 'neighborhoods'). Optionally one borough. Each list carries its rule, its true count and its first members. State the rule; don't call members 'best' or 'hidden gems'.
- `find-places` — Use this when a family asks what's in one NYC neighborhood, or where a kind of place is in it: a library, playgrounds, subway stations or bus stops, coffee, groceries, restaurants, things to do, family resources (rec centers, farmers markets), healthcare (pediatricians, urgent care, hospitals, dentists) or public, charter and private schools. Searches every place we map inside the neighborhood's boundary by name, street or kind (query), or lists every place of one kind (kind); with neither, returns what kinds of places are there, with counts. Results are grouped by category, with a link to each school's page and to the neighborhood's page, which lists every place. Use the nta_code from search-neighborhoods or find-neighborhood.
- `get-admissions-methods` — Get the full NYC school-admissions reference: every entry point from 3-K and Pre-K through Kindergarten, Gifted & Talented, and the 8 middle/high admission methods (SHSAT, audition, screened, Ed-Opt, limited unscreened, unscreened, zoned, charter lottery) — each with its typical application window. The methods carry a live program count plus a special_badge naming any extra ask beyond the standard application (an audition, a test, an open-house sign-in), null when there is none; optional band/governance filters narrow the count. Knowledge only.
- `explain-admissions-method` — Explain one NYC admission entry point or method in full — the plain-English explainer, its typical deadline window, and, when the method asks a family for more than a ranked application, exactly what it requires (audition or portfolio, SHSAT registration, a demonstrated-interest sign-in, a private school's own test/interview track) plus the extra paperwork that comes with it. Pass a method key (shsat, audition, screened, ed_opt, limited_unscreened, unscreened, zoned, charter_lottery, private_admissions, prek_universal) or an early entry point (three-k, pre-k, kindergarten, gifted-talented).
- `get-admissions-demand` — Use this when a family asks how a specific school admits students or how hard it is to get in. Returns the school's admission method in full (what it asks of a family, and when), its programs with seats and who may apply, and — where NYC MySchools publishes them — seats vs. applicants per program with a competitiveness band that is Motley's own, not a DOE rating. Most middle schools, and all charter and private schools, have no applicant counts. The counts describe the whole applicant pool: never turn them into a family's odds, chance or percentage, even when asked for one. Needs a DBN; resolve a name with find-school first.
- `get-admissions-timeline` — Use this when a family asks what's coming for their child — which NYC school applications (3-K, Pre-K, Kindergarten, Gifted & Talented, middle school, high school) they'll make, and when. Pass each child's current grade, or an age (0–5) for a child not in school yet. Returns every transition in the next two years with the city's published dates where it has announced them, and the typical window where it hasn't. Never present a typical window as a date.

## Resources and prompts

- `motley://neighborhoods` — NYC neighborhoods catalog
- `motley://admissions/methods` — NYC admissions reference
- `ui://motley/cards-v1.html` — Result cards for rank-neighborhoods, find-places, find-school and browse-schools: an MCP Apps view a host renders inline with the tool's result.
- `motley://neighborhood/{nta_code}` — Neighborhood summary
- `motley://admissions/method/{key}` — Admission method explainer
- Prompt `compare-neighborhoods-for-young-family` — Compare NYC neighborhoods for a young family in a given borough, weighing safety, schools, and family density.
- Prompt `neighborhood-fit-for-toddler` — Assess whether one NYC neighborhood is a good fit for a family with a toddler (playgrounds, safety, nearby activities).
- Prompt `kindergarten-admissions-timeline` — Walk a parent through how NYC admissions works for a given early entry point (defaults to kindergarten) — the process and the typical application window.

## Ownership

Maintained by Motley / HypnoLabs.
