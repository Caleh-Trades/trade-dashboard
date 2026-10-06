# Trading performance dashboard

Static single-page dashboard for a long-only spot day-trading assistant. It reads `data.json` in the browser and shows percentages, R multiples, and market prices. It does not show balances, account IDs, or other identifying account data.

The assistant should overwrite `data.json` in the repository root after each trade or cycle and push that change. Reload the page to see the new file. While the page is open it also refetches about once a minute.

## Enable GitHub Pages

1. Open this repository on GitHub.
2. Go to **Settings > Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Set the branch to **main** and the folder to **/ (root)**.
5. Save.

The site is published at `https://caleh-trades.github.io/trade-dashboard/`. `.nojekyll` is included so Pages serves `data.json` as a real file.

The page loads `data.json` with `fetch`, so it has to be served over HTTP. Opening `index.html` as a local file will fail that request. GitHub Pages serves it correctly after the steps above. For a local check, run a static server in this folder (for example `python3 -m http.server`).

## `data.json`

Top-level keys, and only these keys, are:

| Key | Meaning |
| --- | --- |
| `updated` | ISO 8601 timestamp of the last write. Shown in Hong Kong Time (HKT, `Asia/Hong_Kong`). |
| `stats` | Lifetime headline figures. |
| `today` | Current session P&L against the daily loss cap. |
| `equity_curve` | Cumulative return over time. |
| `open_positions` | Positions still open. |
| `closed_trades` | Finished trades. The page lists them newest first. |
| `strategies` | Per-strategy scorecards. |
| `paper` | In-window vs out-of-window paper comparison. |

`stats`:

| Field | Meaning |
| --- | --- |
| `trades`, `wins`, `losses` | Counts. |
| `win_rate` | Win rate as a percentage, not a fraction. `66.7` means 66.7%. Use `null` when it is undefined. |
| `avg_r` | Average R multiple. `null` when undefined. |
| `profit_factor` | Gross profit divided by gross loss. `null` when undefined. |
| `total_return_pct` | Cumulative return in percent points. `1.25` means +1.25%. |
| `max_drawdown_pct` | Largest peak-to-trough decline as a positive percent. `0.4` means 0.40%. |

`today`:

| Field | Meaning |
| --- | --- |
| `date` | Session date, `YYYY-MM-DD`. |
| `pnl_pct` | Today's P&L in percent points. |
| `loss_cap_pct` | Daily loss cap in percent points. `1.0` means 1%. |
| `trades_taken` | Trades taken today. |

Every `*_pct` field is already in percent points (`1.0` = 1%). `null` numbers render as an em dash. Do not add balances, account IDs, API keys, or other identifying fields. The repository is public.

### `equity_curve` items

```json
{ "t": "2026-10-06T19:20:00+08:00", "return_pct": 0.0 }
```

`t` is an ISO 8601 timestamp. `return_pct` is the cumulative return at that time.

### `open_positions` items

```json
{
  "id": "BTC-1",
  "symbol": "BTCUSDC",
  "side": "LONG",
  "strategy": "VWAP pullback",
  "opened": "2026-10-06T20:30:00+08:00",
  "entry": 63540.5,
  "stop": 63110.0,
  "targets": [63980.0, 64420.0],
  "risk_pct": 0.35,
  "last_price": 63690.2,
  "unrealized_r": 0.35
}
```

Prices are market prices. `risk_pct` is the position risk in percent points. `unrealized_r` is the open result in R.

### `closed_trades` items

```json
{
  "id": "BTC-1",
  "symbol": "BTCUSDC",
  "side": "LONG",
  "strategy": "VWAP pullback",
  "opened": "2026-10-06T09:40:00+08:00",
  "closed": "2026-10-06T10:20:00+08:00",
  "entry": 62880.0,
  "exit": 63410.0,
  "r": 1.52,
  "pnl_pct": 0.48,
  "exit_reason": "T1"
}
```

`r` is the realized R multiple. `pnl_pct` is the trade's contribution in percent points. `exit_reason` is a short label such as `T1` or `stop`.

### `strategies` items

```json
{
  "name": "VWAP pullback",
  "trades": 0,
  "win_rate": null,
  "avg_r": null,
  "profit_factor": null,
  "status": "unproven"
}
```

`status` is a short label. `unproven` is the expected starting value.

### `paper`

`in_window` and `out_of_window` each use the same four fields:

```json
{ "trades": 0, "win_rate": null, "avg_r": null, "profit_factor": null }
```

In-window setups are the ones inside the trading window. Out-of-window setups are the paper comparison.
