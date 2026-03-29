# Kalshi Market Data

Real-time market microstructure data collected from [Kalshi](https://kalshi.com) sports prediction markets via the Kalshi WebSocket API. Data is collected at **1-second granularity** and includes prices, normalized order flow imbalance (OFI), full order book depth, and raw trade feed.

---

## Supported Markets

| Ticker Prefix | League |
|---|---|
| `KXNBAGAME` | NBA |
| `KXNCAAMBGAME` | NCAA Men's Basketball |
| `KXNCAAWBGAME` | NCAA Women's Basketball |
| `KXNHLGAME` | NHL |
| `KXMLBSTGAME` | MLB |
| `KXEPLGAME` | English Premier League |
| `KXUCLGAME` | UEFA Champions League |
| `KXLALIGAGAME` | La Liga |
| `KXBUNDESLIGAGAME` | Bundesliga |
| `KXSERIEAGAME` | Serie A |
| `KXLIGUE1GAME` | Ligue 1 |
| `KXMLSGAME` | MLS |
| `KXLIGAMXGAME` | Liga MX |
| `KXEFLCHAMPIONSHIPGAME` | EFL Championship |

---

## Repository Structure

```
Kalshi-Market-Data/
└── {SERIES_TICKER}/
    ├── prices/
    │   └── {TICKER}.csv
    ├── depth/
    │   └── DEPTH_{TICKER}.csv
    └── trades/
        └── TRADES_{TICKER}.csv
```

Each game is identified by its full Kalshi market ticker (e.g. `KXNBAGAME-26MAR05BOSDET-BOS`). Data collection begins ~15 minutes before tipoff/kickoff and runs through market settlement.

---

## Data Schema

### `prices/{TICKER}.csv` — 1-second price and flow summary

| Column | Description |
|---|---|
| `ts` | Unix timestamp (seconds) |
| `price_yes` / `price_no` | Best yes/no prices in cents (0–100) |
| `best_ask` / `best_bid` | Top of book ask and bid |
| `spread` | `best_ask - best_bid` |
| `{W}sec_norm_OFI` | Normalized Order Flow Imbalance over window W ∈ {3, 5, 30} seconds. Range: [-1, 1] |
| `{W}sec_yes_vol` / `{W}sec_no_vol` / `{W}sec_total_vol` | Directional and total volume over window W |
| `1sec_*_vol` / `60sec_*_vol` | 1-second and 60-second volume snapshots |
| `ask_total_depth` / `bid_total_depth` | Total liquidity across all visible levels |
| `depth_imbalance` | Signed imbalance between ask and bid depth |
| `ask_depth_change_3s` / `ask_depth_change_10s` | Ask-side depth delta over 3s and 10s windows |
| `ask_levels` / `bid_levels` | Number of active price levels on each side |

### `depth/DEPTH_{TICKER}.csv` — Full order book ladder (1-second snapshots)

| Column | Description |
|---|---|
| `ts` | Unix timestamp (seconds) |
| `best_ask` / `best_bid` | Top of book prices |
| `ask_price_{0..10}` / `ask_size_{0..10}` | Price and size at each ask level (level 0 = best) |
| `bid_price_{0..10}` / `bid_size_{0..10}` | Price and size at each bid level (level 0 = best) |
| `ask_depth_10` / `bid_depth_10` | Total size summed across top 10 levels on each side |

### `trades/TRADES_{TICKER}.csv` — Raw trade feed

| Column | Description |
|---|---|
| `Sequence id` | Kalshi sequence number |
| `Time sec` | Unix timestamp (seconds) |
| `Datetime` | Human-readable timestamp |
| `Yes px` / `No px` | Trade price in cents |
| `Side taken` | Aggressor side (`yes` or `no`) |
| `Size` | Number of contracts traded |

---

## Key Concept

**Order Flow Imbalance (OFI)** measures the directional aggression of takers in the market. A value of +1 means all volume in the window was aggressive buying (yes side), -1 means all aggressive selling. OFI is normalized by total volume to make it comparable across windows and games.

---

## Data Collection

Data is collected using a custom multi-ticker WebSocket aggregator that subscribes to both `trade` and `orderbook_delta` channels simultaneously. The order book state is maintained in memory via snapshot + delta reconstruction. All three CSVs are written on a 1-second clock-aligned tick.
