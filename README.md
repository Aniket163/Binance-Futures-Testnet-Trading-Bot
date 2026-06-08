# Binance-Futures-Testnet-Trading-Bot
A clean, production-structured Python CLI application for placing orders on the Binance USDT-M Futures Testnet.
# Binance Futures Testnet Trading Bot

A clean, production-structured Python CLI application for placing orders on the **Binance USDT-M Futures Testnet**.

---

## Project Structure

```
trading_bot/
├── bot/
│   ├── __init__.py          # Package exports
│   ├── client.py            # Low-level Binance REST client (signing, HTTP, error mapping)
│   ├── orders.py            # Order placement logic + OrderResult dataclass
│   ├── validators.py        # Input validation (symbol, side, type, qty, price)
│   └── logging_config.py   # Rotating file + console log handlers
├── cli.py                   # CLI entry point (argparse sub-commands)
├── logs/
│   └── trading_bot.log      # Auto-created on first run
├── .env.example             # Rename to .env and fill in credentials
├── requirements.txt
└── README.md
```

---

## Setup

### 1. Clone / unzip the project

```bash
cd trading_bot
```

### 2. Create a virtual environment

```bash
python -m venv venv
source venv/bin/activate        # macOS / Linux
venv\Scripts\activate.bat       # Windows CMD
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Get Binance Futures Testnet credentials

1. Visit <https://testnet.binancefuture.com>
2. Log in with your GitHub account
3. Click **"Generate HMAC_SHA256 Key"** under *API Key*
4. Copy the **API Key** and **Secret Key**

### 5. Configure credentials

```bash
cp .env.example .env
```

Edit `.env`:

```
BINANCE_API_KEY=your_testnet_api_key_here
BINANCE_API_SECRET=your_testnet_api_secret_here
```

> **Note:** Never commit `.env` to version control.

---

## How to Run

All commands go through `cli.py`.

### Place a MARKET order

```bash
# BUY 0.001 BTC at market price
python cli.py place --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001

# SELL 0.002 ETH at market price
python cli.py place --symbol ETHUSDT --side SELL --type MARKET --quantity 0.002
```

### Place a LIMIT order

```bash
# SELL 0.001 BTC with a limit price of 72,000 USDT (GTC)
python cli.py place --symbol BTCUSDT --side SELL --type LIMIT --quantity 0.001 --price 72000

# BUY 0.005 ETH with limit 3200, IOC
python cli.py place --symbol ETHUSDT --side BUY --type LIMIT --quantity 0.005 --price 3200 --time-in-force IOC
```

### Place a STOP_MARKET order *(bonus order type)*

```bash
# Trigger a market BUY if price drops to 65,000
python cli.py place --symbol BTCUSDT --side BUY --type STOP_MARKET --quantity 0.001 --stop-price 65000
```

### View account balances

```bash
python cli.py account
```

### List open orders

```bash
python cli.py open-orders                   # all symbols
python cli.py open-orders --symbol BTCUSDT  # filtered
```

### Cancel an order

```bash
python cli.py cancel --symbol BTCUSDT --order-id 3865864258
```

### Adjust log verbosity

```bash
python cli.py --log-level DEBUG place --symbol BTCUSDT --side BUY --type MARKET --quantity 0.001
```

---

## Sample Output

```
  Sending order to Binance Futures Testnet…

────────────────────────────────────────────────────────────
  ORDER REQUEST SUMMARY
────────────────────────────────────────────────────────────
  Symbol             BTCUSDT
  Side               BUY
  Type               MARKET
  Quantity           0.001
────────────────────────────────────────────────────────────
  ORDER RESPONSE
────────────────────────────────────────────────────────────
  Order ID           3865864100
  Client OID         testnet_abc123
  Symbol             BTCUSDT
  Side               BUY
  Type               MARKET
  Status             FILLED
  Original Qty       0.001
  Executed Qty       0.001
  Avg Price          69421.50000
────────────────────────────────────────────────────────────
  ✅  Order placed successfully!
────────────────────────────────────────────────────────────
```

---

## Logging

Log files are written to `logs/trading_bot.log` automatically.

- **Console**: INFO level and above (configurable via `--log-level`)
- **File**: DEBUG level (all requests, responses, errors)
- Log files rotate at **5 MB**, keeping the last 3 backups

```
2025-06-08T10:12:03 | INFO     | trading_bot | Logging initialised
2025-06-08T10:12:04 | INFO     | trading_bot.orders | Placing order: symbol=BTCUSDT side=BUY type=MARKET qty=0.001 price=—
2025-06-08T10:12:04 | DEBUG    | trading_bot.client | → POST /fapi/v1/order  params={...}
2025-06-08T10:12:04 | DEBUG    | trading_bot.client | ← HTTP 200  body={...}
2025-06-08T10:12:04 | INFO     | trading_bot.orders | Order placed successfully: orderId=3865864100 status=FILLED executedQty=0.001
```

---

## Error Handling

| Scenario | Behaviour |
|---|---|
| Missing `--price` for LIMIT | Validation error printed; nothing sent to API |
| Invalid symbol | Binance API error `(-1121)` caught and displayed |
| Network timeout | `BinanceNetworkError` caught, friendly message shown |
| Bad credentials | Binance API error `(-2015)` caught and displayed |
| `quantity <= 0` | Validation rejects before any network call |

---

## Architecture

```
cli.py              ← argparse sub-commands; human-readable output
    │
    └── bot/orders.py    ← validates inputs; builds API params; returns OrderResult
            │
            └── bot/client.py    ← signs requests; raw HTTP; maps errors to exceptions
```

- **No circular imports** — client knows nothing about orders; orders know nothing about CLI
- **`OrderResult` dataclass** decouples the API response shape from the CLI display logic
- **`validators.py`** is stateless and pure — easy to unit-test independently

---

## Assumptions

1. The bot targets **USDT-M Futures Testnet** only (`https://testnet.binancefuture.com`).
2. Credentials are loaded from environment variables / `.env` — not hardcoded.
3. Default `timeInForce` is `GTC`; override with `--time-in-force`.
4. `STOP_MARKET` is implemented as the bonus order type.
5. Quantity / price precision is passed as-is; users are responsible for respecting Binance's `LOT_SIZE` and `PRICE_FILTER` rules for the chosen symbol.
6. Python 3.9+ is assumed (uses `dict[str, Any]` type hints).

---

## Dependencies

| Package | Purpose |
|---|---|
| `requests` | HTTP client for Binance REST API |
| `python-dotenv` | Load `.env` credentials file |

Both are standard, minimal, and widely audited.
