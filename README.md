# Order Book Market Making Bot

An automated market making system that quotes both a bid and an ask price on live order book data, manages inventory risk as orders get filled, and adjusts its quotes in real time to reduce adverse selection.

This project runs on exchange testnet data (paper trading), not real funds, so it can be built, tested, and demonstrated without capital or KYC requirements.

---

## Table of Contents

- [Motivation](#motivation)
- [Data Source](#data-source)
- [Methodology](#methodology)
- [What This Project Analyzes](#what-this-project-analyzes)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Results](#results)
- [Limitations](#limitations)
- [Future Work](#future-work)
- [License](#license)

---

## Motivation

Market making is the core function of most proprietary trading desks: continuously quoting a bid and an ask, earning the spread on balanced flow, while managing the risk of accumulating a one-sided position when the market moves against you.

This project builds and runs an actual quoting engine to demonstrate:

- How to generate and update live bid/ask quotes based on market conditions
- How to manage inventory risk so the bot does not accumulate unbounded exposure
- How real market making performance is measured, beyond just "did it make money"

It is a hands-on demonstration of the mechanics behind liquidity provision, which is directly relevant to quant trading and market making roles.

---

## Data Source

Live order book (Level 2) data is pulled from an exchange **testnet**, for example **Binance Futures Testnet**, which provides realistic live market data and a simulated trading account funded with test capital. No real funds or KYC are required.

Data collected includes:

| Data Point | Purpose |
|------------|---------|
| Best bid and ask prices | Base reference for quote placement |
| Order book depth (top N levels) | Estimating fill probability and slippage |
| Trade fills on the bot's own orders | Tracking inventory and realized P&L |
| Timestamped mid-price | Benchmarking quote performance |

---

## Methodology

1. **Connect to the exchange testnet** via WebSocket to receive live order book updates.
2. **Calculate a reference price**, typically the mid-price between the best bid and best ask.
3. **Generate quotes** by placing a bid below and an ask above the reference price, separated by a spread wide enough to cover expected costs and risk.
4. **Skew quotes based on inventory** — using an approach inspired by the Avellaneda-Stoikov market making model, the bot shifts its quotes when it is holding too much or too little inventory, making it more attractive to trade back toward a neutral position.
5. **Track fills** as orders get executed, updating current inventory and cash position.
6. **Enforce inventory risk limits** — if the position size exceeds a defined threshold, the bot stops quoting on that side or quotes far more aggressively to reduce risk.
7. **Log every quote update, fill, and inventory change** with timestamps for later analysis.
8. **Run continuously over an extended period** (days, not minutes) to gather a meaningful sample of fills and market conditions.

---

## What This Project Analyzes

- **Fill rate** — how often the bot's quotes actually get executed
- **Realized P&L** — profit or loss from the spread captured, net of any adverse price moves
- **Inventory over time** — how well the skewing logic keeps the position near neutral
- **Adverse selection** — whether fills tend to happen right before the market moves against the bot's new position
- **Spread and skew sensitivity** — how performance changes when the spread width or the inventory skew aggressiveness is adjusted

---

## Tech Stack

- **Python** — core bot logic, quote generation, and risk management
- **Exchange WebSocket and REST API** (e.g., Binance Futures Testnet) — live market data and order placement
- **A data storage layer** such as CSV files or a lightweight local database for logging quotes, fills, and inventory
- **Matplotlib or Plotly** — visualizing P&L, inventory, and fill activity over time

---

## Project Structure

```
orderbook-market-maker/
│
├── data_feed/
│   └── orderbook_stream.py     # Connects to exchange, streams live order book data
│
├── strategy/
│   └── quote_engine.py         # Generates bid/ask quotes and applies inventory skew
│
├── execution/
│   └── order_manager.py        # Places, updates, and cancels orders on the testnet
│
├── risk/
│   └── inventory_limits.py     # Enforces maximum position size and risk rules
│
├── logging/
│   └── log_activity.py         # Records quotes, fills, and inventory changes
│
├── analysis/
│   └── performance_report.py   # Summarizes P&L, fill rate, and inventory behavior
│
├── results/
│   ├── activity_log.csv
│   └── charts/
│
├── requirements.txt
└── README.md
```

---

## How to Run

1. **Clone the repository**
   ```
   git clone <repository-url>
   cd orderbook-market-maker
   ```

2. **Set up a virtual environment**
   ```
   python -m venv venv
   source venv/bin/activate
   ```

3. **Install dependencies**
   ```
   pip install -r requirements.txt
   ```

4. **Create a testnet account and API key** on the chosen exchange testnet, and add the credentials to a local configuration file (never commit API keys to the repository).

5. **Run the market making bot**
   ```
   python execution/order_manager.py
   ```

6. **Run the performance report** after the bot has been running for a period of time
   ```
   python analysis/performance_report.py
   ```

7. **View results** in the `results/` folder, including the activity log and generated charts

---

## Results

A summary of:

- Total quotes placed and total fills received over the monitoring period
- Fill rate (percentage of quotes that resulted in a trade)
- Realized P&L over the monitoring period
- Maximum and average inventory held
- Charts showing P&L, inventory, and quote activity over time

*(To be filled in once a live monitoring run is complete.)*

---

## Limitations

- Runs on testnet data, so fills and price behavior may not perfectly match a live, real-money order book with other real participants
- Does not account for exchange latency beyond what the testnet itself introduces
- The quoting model is simplified relative to models used by professional market makers, which often incorporate additional signals such as order flow toxicity and short-term price prediction

---

## Future Work

- Incorporate a short-term price prediction signal (e.g., order flow imbalance) into quote placement, rather than relying only on the current mid-price
- Test performance across different volatility regimes
- Extend the bot to quote multiple instruments simultaneously and manage inventory risk across a small portfolio

---

## License

This project is for educational and research purposes only and does not constitute financial advice.
