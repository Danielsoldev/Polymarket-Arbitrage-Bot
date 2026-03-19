# Polymarket Arbitrage Bot

Automated hedged arbitrage on Polymarket's 15-minute Up/Down markets (e.g. BTC, ETH, SOL). Uses mean-reversion triggers to enter one side at favorable prices, then hedges the opposite side at `0.98 − firstSidePrice`. Built with TypeScript and Polymarket's CLOB API.

---

## Private Version Bot Profile

![Bot Profile](image/profile.png)

---

## Arbitrage Results

![Arbitrage Result](image/result.png)

---

## Overview


|              |                                                                                                                                                                                                                                                                    |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Strategy** | Hedged mean-reversion: track YES/NO midpoint prices; when one side drops below threshold, trigger buy on reversal, depth discount, or time-based second-side; hedge opposite side at `dynamicThreshold = 1 − firstFillPrice + boost` (e.g. 0.98 − firstSidePrice). |
| **Markets**  | Configurable list (e.g. `btc`, `eth`, `sol`); slugs resolved as `{market}-updown-15m-{startOf15mUnix}` via Gamma API.                                                                                                                                              |
| **Stack**    | TypeScript, Node.js (or Bun), `@polymarket/clob-client`, Ethers.js for allowances/redemption.                                                                                                                                                                      |


---

## Requirements

- Node.js 18+ (or Bun)
- Polygon wallet with USDC and POL (for gas)
- RPC URL for Polygon (e.g. Alchemy) for allowances and redemption

---

## Install

```bash
git clone https://github.com/Danielsoldev/Polymarket-Arbitrage-Bot.git
cd Polymarket-Arbitrage-Bot
npm install
```

---

## Configuration

Copy the example env and set at least `PRIVATE_KEY` and `TRADE_MARKETS`:

```bash
cp .env.example .env
```


| Variable                           | Description                                        | Default                       |
| ---------------------------------- | -------------------------------------------------- | ----------------------------- |
| `PRIVATE_KEY`                      | Wallet private key                                 | required                      |
| `TRADE_MARKETS`                    | Comma-separated markets (e.g. `btc`, `eth`, `sol`) | `btc`                         |
| `TRADE_SHARES`                     | Shares per side per trade                          | `5`                           |
| `TRADE_THRESHOLD`                  | Entry threshold for first side                     | `0.47`                        |
| `TRADE_TICK_SIZE`                  | Price precision                                    | `0.01`                        |
| `TRADE_PRICE_BUFFER`               | Price buffer for execution                         | `0.05`                        |
| `TRADE_WAIT_FOR_NEXT_MARKET_START` | Wait for next 15m boundary before starting         | `false`                       |
| `MAX_BUYS_PER_SIDE`                | Max buys per side per 15m cycle                    | `1`                           |
| `TRADE_MAX_SUM_AVG`                | Max combined avg price guard                       | `0.99`                        |
| `REVERSAL_DELTA`                   | Reversal amount to trigger buy                     | `0.02`                        |
| `TRADE_DEPTH_BUY_DISCOUNT_PERCENT` | Depth discount trigger (%)                         | `0.02`                        |
| `TRADE_DYNAMIC_THRESHOLD_BOOST`    | Boost for second-side threshold                    | `0.04`                        |
| `CHAIN_ID`                         | Chain ID (Polygon)                                 | `137`                         |
| `CLOB_API_URL`                     | CLOB API base URL                                  | `https://clob.polymarket.com` |
| `RPC_URL` / `RPC_TOKEN`            | RPC for allowances/redemption                      | —                             |
| `BOT_MIN_USDC_BALANCE`             | Min USDC to start                                  | `1`                           |
| `LOG_DIR` / `LOG_FILE_PREFIX`      | Log directory and file prefix                      | `logs` / `bot`                |


API credentials are created on first run and stored in `src/data/credential.json`.

---

## Usage

### Run the bot

```bash
npm start
# or: bun src/index.ts
```

### Redemption

```bash
# Auto-redeem resolved markets (holdings file)
npm run redeem:holdings

# Redeem by condition ID
npm run redeem
```

### Balance

```bash
npm run balance:log
```

---

## Development

```bash
npx tsc --noEmit
bun --watch src/index.ts
```

---

## Project Structure


| Path                               | Role                                                                                                                                                  |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `src/index.ts`                     | Entry: credentials, CLOB, allowances, min balance, start CopytradeArbBot                                                                              |
| `src/config/index.ts`              | Loads `.env` and exposes config (chain, CLOB, copytrade, logging)                                                                                     |
| `src/order-builder/copytrade.ts`   | CopytradeArbBot: 15m slug resolution, midpoint polling, three triggers → first-side buy + second-side hedge; state in `src/data/copytrade-state.json` |
| `src/providers/clobclient.ts`      | CLOB client singleton (credentials + PRIVATE_KEY)                                                                                                     |
| `src/security/allowance.ts`        | USDC and CTF approvals                                                                                                                                |
| `src/security/createCredential.ts` | API credential bootstrap                                                                                                                              |
| `src/utils/balance.ts`             | Balance checks / wait gates                                                                                                                           |
| `src/utils/holdings.ts`            | Position persistence for redemption                                                                                                                   |
| `src/utils/console-file.ts`        | Daily file logging                                                                                                                                    |
| `src/data/copytrade-state.json`    | Per-slug state (prices, timestamps, buy counts)                                                                                                       |
| `src/data/token-holding.json`      | Token holdings for redemption (generated)                                                                                                             |


---

## Risk and Disclaimer

Trading prediction markets involves significant risk. This software is provided as-is. Use at your own discretion and only with funds you can afford to lose.

---

## License

ISC