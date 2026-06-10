# AM4 Route Optimizer

A route profitability calculator for [Airlines Manager 4](https://www.airlines-manager.com/). Given a route and operating conditions, it ranks every aircraft in the game by profit per day, recommends a seat configuration, and shows a full cost breakdown per flight.

A self-contained HTML file — no install, no build step, just open in a browser.

> **Authorship:** The code in this project was written entirely by [Claude](https://claude.ai) (Anthropic). This repository was created to share the tool with the AM4 community.

---

## Usage

Download `am4_optimizer.html` and open it in any modern browser. No server required.

---

## Features

- **308 aircraft** across 26 manufacturers (Aerospatiale, Airbus, Antonov, Beechcraft, Boeing, Bombardier, British Aerospace, COMAC, Cessna, Convair, Dassault, Dornier, Embraer, Fokker, Gulfstream, Ilyushin, Irkut, Lockheed, Martin, McDonnell Douglas, Mitsubishi, Saab, Sud Aviation, Sukhoi, Tupolev, Vickers)
- **Easy and Realism mode** — Easy applies the 1.5× speed multiplier; ticket pricing formulas differ between modes
- **Demand-aware seat config** — finds the minimum flights to serve your Y/J/F daily demand, then fills remaining capacity with premium seats to match what the game suggests
- **Runway filter** — enter your airport's runway length to exclude incompatible aircraft
- **Stopover toggle** — doubles effective range for all planes (refueling stop, no other effect)
- **Reputation / load factor** — configurable; drives seat sizing and passengers carried per flight
- Sortable results table with per-flight revenue, cost, and profit breakdown
- Filter by max purchase price

---

## Inputs

| Field | Description |
|---|---|
| Distance (km) | Route distance as displayed in-game |
| Op. Hours/Day | Your daily play session length. Flights counted as **dispatches**, not completions — a 7h59m flight in an 8h session counts twice |
| Reputation % | Your airline reputation; sets the load factor for seat sizing and revenue |
| Runway (ft) | Leave blank for no limit; filters out aircraft that need more runway than your airport provides |
| Y / J / F Demand | Daily passenger demand by class. If set, the optimizer finds the minimum flights to serve it all; if left at zero, fills all seats with economy |
| Max Price ($) | Excludes aircraft above this purchase price |
| Game Mode | Easy or Realism |
| Stopovers | Doubles each aircraft's effective range; no effect on cost or revenue |

---

## Cost Model

Aircraft stats from the **Airplane Efficiency** spreadsheet by Blu Lux Air (see [Attribution](#attribution)).

**Per flight:**

```
fuel_cost   = Fuel (lbs/km) × distance × $0.70/lb
achk_cost   = (A_Check / AC_per_HR) × distance / effective_speed
co2_cost    = CO2 (quotas/pax/km) × actual_pax × distance × $0.13/quota
total_cost  = fuel_cost + achk_cost + co2_cost
```

`effective_speed = aircraft_speed × 1.5` in Easy mode, `× 1.0` in Realism.

Fuel cost is speed-independent (lbs/km cancels speed). A-check cost scales with flight time, so Easy mode has cheaper A-check per trip but more trips per session.

**Ticket prices:**

| Class | Easy | Realism |
|---|---|---|
| Economy (Y) | 0.4d + 170 | 0.3d + 150 |
| Business (J) | 0.8d + 560 | 0.6d + 500 |
| First (F) | 1.2d + 1200 | 0.9d + 1000 |

**Seat config:** J seats occupy 2 Y-slot equivalents; F seats occupy 3. Minimum seats per class are sized using `floor((ceil(demand/flights) − 1) / load_factor) + 1`, which correctly accounts for the game's ceiling rounding when loading passengers. Remaining capacity is filled with F then J seats.

**Not modeled:** aircraft wear / maintenance check costs. The source data's author found these to be effectively random with no fittable linear relationship, so they are excluded rather than estimated.

---

## Optimization Logic

1. For each aircraft with sufficient range (and runway, if set):
2. Find the **minimum number of dispatches** to serve daily demand within seat capacity — fewer flights means lower total cost, since revenue plateaus once all demand is served
3. Dispatch count: `ceil(session_hours × effective_speed / distance)`, counting departures within the session window, not landings
4. Revenue per class: `min(demand/flight, ceil(seats × load_factor))`
5. Exclude aircraft where profit per flight ≤ 0
6. Sort by profit per day

---

## Attribution

Aircraft data from the **Airplane Efficiency** spreadsheet by **Blu Lux Air**, shared with the AM4 community. Manufacturers for the "Other Aircraft" section were re-attributed to their real-world manufacturers (Dornier, Gulfstream, Martin, Convair, Dassault, Mitsubishi, Irkut, Vickers). The cargo variant Bae-146-300QT is excluded as it uses a different data schema.

Airlines Manager 4, aircraft names, and game statistics are the property of Playrion. This is an unofficial fan project, not affiliated with or endorsed by Playrion.

Code written by [Claude](https://claude.ai) (Anthropic).

---

## Version History

| Version | Changes |
|---|---|
| v0.7 | Distance and demand default to empty; reputation defaults to 100%; stopovers checkbox alignment fix |
| v0.6 | Seat fill: remaining capacity filled with premium seats after demand sizing |
| v0.5 | Easy mode 1.5× speed multiplier; stopover checkbox (2× range) |
| v0.4 | Ceiling rounding for passenger loading; corrected seat sizing formula |
| v0.3 | Reputation % input; runway length filter; maintenance cost disclaimer |
| v0.2 | Rebuilt aircraft dataset from source; removed phantom entries; fixed manufacturer labels |
| v0.1 | Initial release |

---

## License

MIT — see [LICENSE](LICENSE). Aircraft data and game content remain the property of their respective owners.
