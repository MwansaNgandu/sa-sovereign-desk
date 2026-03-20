# SA Sovereign Desk — Macro Intelligence Dashboard

A single-file, browser-based macro dashboard built for South Africa sovereign strategy analysis. Designed to replicate the kind of morning briefing tool used by institutional macro desks — covering fixed income, FX, commodities, equities, load shedding, China demand, freight, and a rules-based macro regime classifier.

Built as a learning and research project by a UCT finance student. All data sourcing, methodology, and limitations are documented in the Data Dictionary section within the dashboard itself.

---

## Screenshot

> _Open `sa-macro-tier1.html` locally to see the live dashboard._

---

## What It Does

| Section | Description |
|---|---|
| **Ticker Strip** | Live USD/ZAR, EUR/ZAR, GBP/ZAR · Brent, Gold, Platinum, Palladium · JSE Top 40 proxy |
| **Macro Regime Classifier** | Rules-based engine scoring 6 signal dimensions → classifies into one of 5 SA macro regimes |
| **SAGB Yield Curve** | Full par curve 3M → 30Y · 2s10s and 2s30s spreads · real yield vs CPI |
| **USD/ZAR Vol Surface** | Indicative implied vol heatmap · tenor × delta grid |
| **Sovereign CDS** | SA 5Y CDS vs 7 EM peers · bar chart comparison |
| **Macro Snapshot** | Repo rate · CPI · 10Y yield · credit ratings · load shedding · CA balance · fiscal deficit · FX reserves |
| **Terms of Trade** | Export commodity basket vs import basket ratio · live commodity price inputs |
| **PMI Composite** | SA, China, US, Eurozone, Global, EM · 9-month trend chart |
| **Freight & Container** | BDI · SCFI · Cape of Good Hope transits · Durban TEU · Richards Bay coal |
| **China Demand Complex** | NBS PMI · steel production · iron ore inventories · copper imports · property starts |
| **Commodity/ZAR Correlation** | Rolling Pearson r · accumulates from live prices stored in localStorage |
| **JSE Equities Watch** | 7 tickers · live price · analyst consensus · recent headlines |
| **Macro News Feed** | 14 deduplicated headlines across 7 SA macro query categories |
| **Data Dictionary** | Every data series documented: source, type, update frequency, live fetch status |

---

## Setup

This is a single HTML file. No build step, no server, no npm. Just open it in a browser.

### 1. Get API Keys

You need three free API keys:

| Service | Used For | Sign Up |
|---|---|---|
| **Finnhub** | Commodities (Brent, Gold, Platinum, Palladium), JSE Top 40 proxy (EZA), analyst consensus | [finnhub.io](https://finnhub.io) |
| **EskomSePush** | Live load shedding stage | [developer.sepush.co.za](https://developer.sepush.co.za) |
| **NewsAPI** | Macro news feed + per-stock headlines | [newsapi.org](https://newsapi.org) |

> **Note on NewsAPI:** The free tier only works from `localhost` or local files — not from a hosted/deployed URL. This dashboard is designed to run locally, so the free tier is sufficient.

> **Note on EskomSePush:** Free tier allows 50 calls/day. The dashboard calls this endpoint only on load and on the 60-second auto-refresh, so daily usage stays well within limits under normal use.

FX rates (USD/ZAR, EUR/ZAR, GBP/ZAR) are fetched from **Yahoo Finance** via a CORS proxy cascade — no key or sign-up required.

### 2. Add Your Keys

Open `sa-macro-tier1.html` in any text editor and find these three lines near the top of the `<script>` block (around line 678):

```js
const FINNHUB_KEY  = 'YOUR_FINNHUB_API_KEY';
const ESP_KEY      = 'YOUR_ESKOMSEPUSH_API_KEY';
const NEWSAPI_KEY  = 'YOUR_NEWSAPI_KEY';
```

Replace the placeholder strings with your actual keys. Save the file.

### 3. Open in Browser

Open `sa-macro-tier1.html` directly in Chrome or Firefox. All data fetches on load. The dashboard auto-refreshes every 60 seconds.

---

## Data Sources & Methodology

### What Is Actually Live

| Data | Source | Notes |
|---|---|---|
| USD/ZAR, EUR/ZAR, GBP/ZAR | Yahoo Finance (`USDZAR=X` etc.) via CORS proxy | Intraday · real % change |
| Brent, Gold, Platinum, Palladium | Finnhub | Intraday |
| JSE Top 40 (proxy) | Finnhub · EZA ETF | USD-listed ETF proxy, not the index directly |
| JSE Equities (7 tickers) | Yahoo Finance v8 API via CORS proxy | NPN.JO, PRX.AS, SBK.JO, FSR.JO, SOL.JO, AGL.L, MTN.JO |
| Analyst consensus | Finnhub recommendations endpoint | |
| Load shedding stage | EskomSePush API | |
| News feed | NewsAPI.org | 7 query categories, deduplicated |
| GDP growth, CA balance, Debt/GDP, FX reserves | World Bank API | Free · annual · no key required |
| Commodity/ZAR correlation | Computed from localStorage price history | Seeds from hardcoded values; builds real Pearson r over time |

### What Is Hardcoded / Static

| Data | Source | Last Updated |
|---|---|---|
| SARB Repo Rate | SARB MPC decision | Nov 2025 |
| CPI Inflation | Stats SA | Feb 2026 |
| SAGB Yield Curve shape | SARB benchmark yields (manually sourced) | Mar 2026 |
| Sovereign CDS spreads | IHS Markit / public sources (indicative) | Mar 2026 |
| Credit ratings | Moody's / S&P / Fitch | Nov 2025 (S&P upgrade) |
| USD/ZAR vol surface | Indicative — typical market structure | Not refreshed |
| PMI composite | Absa / NBS / ISM / HCOB releases | Mar 2026 |
| China demand indicators | NBS / Mysteel (indicative) | Mar 2026 |
| BDI / SCFI freight | Baltic Exchange / SSE (indicative) | Mar 2026 |
| Budget deficit | National Treasury Budget Review 2026 | Mar 2026 |
| Terms of Trade (history) | Constructed series — ratio uses live commodity prices | Mar 2026 |

> Static values are clearly labelled with **Static** badges in the dashboard UI and fully documented in the built-in Data Dictionary footer.

### CORS Proxy Cascade

Yahoo Finance calls go through a cascade of four public CORS proxies. The dashboard tries each in order and moves to the next if one fails:

1. `corsproxy.io`
2. `api.allorigins.win`
3. `thingproxy.freeboard.io`
4. `api.codetabs.com`

This makes the dashboard resilient to any single proxy being unavailable.

### Macro Regime Classifier

A rules-based signal engine scores six dimensions on a [-1, +1] scale:

| Dimension | Input |
|---|---|
| FX Momentum | Live USD/ZAR daily change |
| Yield Curve Shape | 2s10s spread from SAGB curve |
| Commodity Complex | Average daily change across export commodities |
| CDS / Credit | SA 5Y CDS level (hardcoded) |
| PMI / Growth | SA Absa PMI (hardcoded) |
| Energy Constraint | Live EskomSePush stage |

Scores are mapped to five regime classifications: **Risk-On · Commodity Boom**, **Stagflation Stress**, **EM Risk-Off / External Shock**, **Domestic Fiscal Stress**, and **Goldilocks · Soft Landing**.

> **Important:** The regime output is a simplified analytical framework, not a live signal. Two of the six inputs (CDS, PMI) are hardcoded and require manual updates. Treat regime classification as indicative context, not a trading signal.

---

## Limitations

- **Vol surface** is indicative only. Live OTC FX implied vol requires Bloomberg OVDV or Refinitiv (paid).
- **CDS spreads** are sourced manually. Live sovereign CDS requires IHS Markit or ICE Data Services (paid).
- **PMI data** is owned by S&P Global Markit. No free API exists.
- **SAGB yield curve** uses manually sourced benchmark yields as a baseline. A SARB page scrape is attempted on load but is fragile and frequently fails due to CORS proxy limitations on government sites.
- **NewsAPI** free tier restricts to `localhost` / local file usage and returns articles up to ~1 month old.
- The dashboard is not designed for deployment on a public server. It is a local analytical tool.

---

## Upgrading to Live Data

If you have access to institutional data platforms, the following upgrades are possible:

| Currently Static | Upgrade Path |
|---|---|
| SAGB yield curve | Refinitiv LSEG Data Library or Bloomberg B-PIPE |
| CDS spreads | ICE Data Services or Bloomberg CDSW |
| Vol surface | Bloomberg OVDV page or Refinitiv FX vol analytics |
| PMI data | Refinitiv / S&P Global Markit API |
| Freight (BDI/SCFI) | Baltic Exchange data API or Quandl |
| China demand | Wind Financial Terminal or Mysteel (both paywalled) |

UCT students may be able to access Refinitiv Eikon/Datastream and IRESS Expert through the UCT library database portal — check with the library for API key access.

---

## Tech Stack

- Vanilla HTML, CSS, JavaScript — no frameworks, no build step
- [Chart.js 4.4.1](https://www.chartjs.org/) — yield curve, PMI trend, CDS bar chart, ToT index, correlation matrix
- [IBM Plex Sans + IBM Plex Mono + Playfair Display](https://fonts.google.com/) — via Google Fonts
- localStorage — rolling price history for correlation computation

---

## Disclaimer

This dashboard is for educational, research, and analytical purposes only. It does not constitute investment advice and does not represent a solicitation or offer to buy or sell any financial instrument. Hardcoded and indicative values may be outdated and should not be relied upon for trading or investment decisions. For institutional-grade data, subscribe to Bloomberg, Refinitiv LSEG, or ICE Data Services.

---

## Licence

MIT — free to use, modify, and distribute with attribution.
#   s a - s o v e r e i g n - d e s k  
 