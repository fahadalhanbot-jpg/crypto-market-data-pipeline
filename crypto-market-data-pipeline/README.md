# 📊 Crypto Market Data Pipeline

An automated **daily ETL pipeline** built with [n8n](https://n8n.io) that fetches live cryptocurrency market data, calculates technical indicators, logs everything to Google Sheets as a time-series history, and emails alerts whenever a coin makes a significant move.

No paid API keys required — it runs entirely on the free CoinGecko public API.

---

## 🚀 What it does

Every day at 9:00 AM the pipeline automatically:

1. **Extracts** live prices for the top coins (BTC, ETH, SOL, ADA, XRP) from the CoinGecko API.
2. **Transforms** the raw data into useful indicators — volatility, 24h price position, momentum, and a BULLISH / BEARISH / NEUTRAL signal.
3. **Loads** every snapshot into a Google Sheet, building a growing time-series dataset.
4. **Alerts** you by email (with a clean HTML table) whenever one or more coins move more than **±5%** in 24 hours.

---

## 🧱 Architecture

```
┌──────────────┐   ┌────────────────────┐   ┌─────────────────────┐
│  Daily 9 AM  │──▶│ Fetch Crypto Prices │──▶│ Calculate Indicators │
│  (Schedule)  │   │  (CoinGecko API)    │   │   (Code / JS)        │
└──────────────┘   └────────────────────┘   └──────────┬──────────┘
                                                        │
                          ┌─────────────────────────────▼─────────────┐
                          │            Save to History Sheet           │
                          │              (Google Sheets)               │
                          └─────────────────────────────┬─────────────┘
                                                        │
                              ┌──────────────────────────▼──────────┐
                              │           Filter Big Movers          │
                              │   (keep coins with |Δ24h| ≥ 5%)      │
                              └──────────────────────────┬──────────┘
                                                        │
                                       ┌─────────────────▼─────────┐
                                       │      Email Big Movers      │
                                       │          (Gmail)           │
                                       └────────────────────────────┘
```

---

## 📐 Indicators calculated

| Indicator | Description |
|-----------|-------------|
| **Volatility %** | Intraday range `(high − low) / price` as a percentage |
| **Price Position %** | Where the current price sits within the 24h range (0% = day low, 100% = day high) |
| **Change 24h / 7d %** | Rounded percentage change over each window |
| **Signal** | `BULLISH` (24h > 3% & 7d > 0), `BEARISH` (24h < −3% & 7d < 0), else `NEUTRAL` |

---

## 🛠️ Tech stack

- **n8n** — workflow orchestration
- **CoinGecko API** — free, no-key market data
- **JavaScript (Code node)** — indicator calculation & HTML report generation
- **Google Sheets** — time-series data store
- **Gmail** — alert delivery

---

## ⚙️ Setup

1. **Import the workflow**
   In n8n: *Workflows → Import from File* → select [`workflows/crypto-market-data-pipeline.json`](workflows/crypto-market-data-pipeline.json).

2. **Configure credentials**
   - **Google Sheets** — connect your Google account and create a sheet with these column headers:
     `Date · Name · Symbol · Price (USD) · Market Cap · Volume 24h · Change 24h % · Change 7d % · Volatility % · Price Position % · Signal`
   - **Gmail** — connect your Google account for sending alerts.

3. **Update placeholders**
   - In the *Save to History Sheet* node, replace `YOUR_GOOGLE_SHEET_ID` with your sheet ID.
   - In the *Email Big Movers* node, replace `YOUR_EMAIL@example.com` with your address.

4. **Activate** the workflow. It will now run daily at 9 AM.

> 💡 Want different coins? Edit the `ids` query parameter in the *Fetch Crypto Prices* node (any CoinGecko coin IDs, comma-separated).

---

## 📈 Possible extensions

- Add more coins or a dynamic top-N list
- Push alerts to Slack / Telegram / Discord
- Add RSI, moving averages, or Bollinger bands
- Build a dashboard on top of the Google Sheet history

---

## 📄 License

Released under the [MIT License](LICENSE).
