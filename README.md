# OptiFly — Multi-City Flight Route Optimizer

Finds the cheapest way to visit a fixed set of cities within a travel window, using a
Held–Karp-style dynamic programming solver with flexible stay durations, real flight
pricing/timing data, and a timeline visualization comparing itineraries under different
flexibility settings.

## The problem

Standard flight search tools handle one origin → one destination, or a fixed multi-city
order. Neither answers: *given N cities and a fixed number of days, in what order should
I visit them, and how long should I stay in each, to minimize total flight cost?*

This is a variant of the Traveling Salesman Problem with time windows: visit every city
exactly once, return to the origin, respect a total trip horizon, and allow each city's
stay length to flex around a target average (±tolerance days).

## Approach

- **Data**: cost, departure/arrival times, stops, and layovers for every city pair and
  candidate date, pulled from a flight API and filtered for "reasonable" itineraries
  (adaptive max-duration caps by distance, layover limits, stop limits).
- **Solver**: Held–Karp dynamic programming with a bitmask over visited cities. Exact
  optimum, avoids the `n!` brute-force blowup — feasible up to ~10-12 cities.
- **Output**: optimal route + cost for a given tolerance, plus a comparison plot (no
  flexibility vs. ±1 day) showing cost and stay-length trade-offs.

## Results (sample run: MAD → BUD, SCL, CCS, BEY, BER → MAD, 20-day window)

| Tolerance | Total cost | Trip duration |
|---|---|---|
| ±0 days | €2,011.73 | 14 days |
| ±1 day  | €1,832.16 | 15 days |

Allowing just one day of flexibility per city cut the total cost by **€179.57 (~9%)**,
with the route reordering itself and stays becoming more balanced across cities.

## Setup

Requires Python 3.10+ (uses `zoneinfo`) and `matplotlib`.

```bash
pip install requests matplotlib
```

Edit the `cities`, `start_date`, `avg_days`, and `TOTAL_DAYS` variables in the config
cell to run it against a different itinerary.

## ⚠️ Note on the flight data API

This project was built against Amadeus's self-service sandbox API. Amadeus shut down
that entire self-service developer portal on July 17, 2026 — existing API keys were
deactivated and the portal is no longer accessible (Enterprise/paid access is
unaffected). As a result, the notebook's API calls will fail as written; it's kept here
as a reference implementation of the algorithm and data pipeline, not a live demo.
Re-running it would require pointing `fetch_cheapest_reasonable` at a different flight
data source 

## Limitations

- Takes only the single cheapest offer per route/day — ignores other departures,
  durations, or connection quality on the same day.
- Objective is flight cost only; doesn't account for time, comfort, layovers, or
  emissions (could be extended to a multi-objective score).
- No non-flight costs (hotels, ground transport, visas).
- Treats each IATA code as a separate node — doesn't group multi-airport metro areas.
- Single passenger, economy class only.

## Files

- `OptiFly_clean.ipynb` — the full pipeline: API fetch → filtering → DP solver → results
  → timeline plot.
