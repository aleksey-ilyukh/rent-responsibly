[README for Work_Commute_app_documented.md](https://github.com/user-attachments/files/32019632/README.for.Work_Commute_app_documented.md)
# Commute-Aware Rental Affordability Analyzer

A tool for a couple deciding where to live — built around one question most apartment search sites can't answer: not just *"can I afford this rent,"* but *"is this responsible for our household, given our real commutes, our real debts, and what happens if one of our incomes disappears."*

Instead of guessing at a rent budget and hoping it works out, this notebook computes **real, traffic-aware commute boundaries** for two people working in different locations, pulls **live rental listings** inside that boundary, and runs every listing through a full financial model — income, debt, expenses, vehicle fuel cost — to answer three separate questions per listing:

- **Feasible** — does the household's income cover it, month to month?
- **Responsible** — does it stay within standard 36% debt-to-income lending guidance?
- **Safe** — is there still a real cushion if one partner's income disappears entirely?

…plus a separate landlord-screening check (the "3x rent" income rule) under three real lease-signing scenarios, since who signs the lease determines both who a landlord checks and who's legally on the hook if things go wrong.

## Why this exists

Rent affordability calculators typically ask for one number — "what's your income" — and spit out a rule of thumb. That misses almost everything that actually matters for a two-income household:

- Two people, two commutes, two vehicles, two different fuel costs — and traffic isn't the same in every direction.
- Bills get split different ways (50/50 vs. proportional to income), and the "responsible" answer can genuinely differ depending on which one you use.
- A landlord's approval isn't the same question as personal safety — a couple can clear a landlord's income check while one partner is still financially over-exposed if the other stops paying.
- Government rent estimates (like HUD's SAFMR data) are useful for a first pass, but they're zip-code-level statistical estimates, not real, available units.

This project builds all of that logic once, and applies it consistently across every real candidate instead of eyeballing a handful of listings by hand.

## How it works

```
1. Define both people's work locations + required arrival times
        │
        ▼
2. Compute real, traffic-aware commute isochrones (10/20/30/40/50/60-min bands)
   via the Mapbox Directions + Isochrone APIs
        │
        ▼
3. Size a search circle guaranteed to fully contain the real commute boundary,
   and visually confirm it against the isochrone map
        │
        ▼
4. Pull live rental listings inside that circle (via the RentCast API) —
   real addresses, real prices, real bedroom counts
        │
        ▼
5. Filter listings down with true point-in-polygon geometry, keeping only
   the ones ACTUALLY inside the real commute boundary (not just the circle)
        │
        ▼
6. Compute real per-address commute time/cost (traffic-aware, with a
   congestion-cost adjustment) for both people, once per unique building
        │
        ▼
7. Run the full financial model against every listing: income tax
   (state + filing-status aware), debts, expenses, vehicle fuel cost,
   two bill-splitting philosophies, and a worst-case income-loss scenario
        │
        ▼
8. Final report: one row per real listing, with Feasible / Safe /
   Responsible / Landlord Screening verdicts, color-coded and ready to sort
```

## Tech stack

| Purpose | Tool |
|---|---|
| Commute routing & isochrones | [Mapbox](https://www.mapbox.com) Directions + Isochrone APIs (`driving-traffic` profile) |
| Live rental listings | [RentCast](https://www.rentcast.io/api) API |
| Geospatial filtering | `geopandas`, `shapely` |
| Vehicle fuel cost/economy | [fueleconomy.gov](https://www.fueleconomy.gov) public API (EPA data) |
| Data wrangling | `pandas`, `polars` |
| Visualization | `matplotlib`, `contextily` |

## Getting started

### 1. API keys

You'll need two free API keys:

- **Mapbox** — sign up at [mapbox.com](https://www.mapbox.com), generate a token. The free tier is generous for this use case.
- **RentCast** — sign up at [rentcast.io/api](https://www.rentcast.io/api). The free "Developer" plan includes 50 requests/month, which is enough for a handful of searches — this notebook's fetch step can use anywhere from a few requests to several dozen depending on your search radius, so keep an eye on it (see the note on request-efficiency below).

Drop both keys into the `# Packages, Datasets and API` section near the top of the notebook, replacing the placeholder strings.

### 2. Install dependencies

Run the install cells at the top of the notebook (`geopandas`, `shapely`, `contextily`, `matplotlib`), or install everything ahead of time:

```bash
pip install geopandas shapely contextily matplotlib requests pandas polars tqdm numpy
```

### 3. Fill in your own numbers

Everything under **Worksheet** is meant to be edited:

- **Income Calculator** — pay type, rate, hours, state(s), filing status for both people.
- **Expenses** — recurring debts (with real loan terms) and everyday shared/individual expenses.
- **Vehicle** — each person's actual make/model/year, to pull real EPA fuel-economy data.
- **Property Searcher** — both people's work coordinates and required arrival times, and the commute bands you want to search (10/20/30/40/50/60 minutes).

### 4. Run top to bottom

The **Property Searcher** section computes and visually confirms the commute boundary before anything gets pulled from RentCast — worth checking the map output before spending API requests. **Huds Dataset Alternative** pulls and filters real listings. **Report Builder Engine** runs the financial math. **Report** renders the final, color-coded table.

## A note on API request efficiency

RentCast bills per request, not per record — a single request can return up to 500 listings, so search radius matters a lot more than you'd expect. A wider circle than necessary can burn through a large chunk of the free tier in one run. The notebook includes tooling to size the search circle precisely against the real commute boundary (not just a rough guess) and to cache results to CSV so repeat runs don't re-spend requests unnecessarily.

## What this doesn't do

This tool narrows hundreds of real listings down to the ones worth actually looking at — it doesn't (and isn't trying to) replace looking at them. Real listings have real quirks — unit condition, natural light, whether the landlord is any good — that no spreadsheet can capture. The intended workflow is: let this report do the financial screening, then go look at the real, promising candidates yourself.

## Project status

Actively developed as part of a data analytics portfolio. Feedback and forks welcome.
