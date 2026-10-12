# Cross-Repository Architecture Notes

## 1. Evidence and limits

These notes were prepared from Graphify's indexed code snapshots. The indexes reported these commits at the time of inspection:

- TradingAgents composite query: `93ffa43` (composite Graphify workspace result; individual repository freshness may differ).
- Binany AI Trading Bot: `bed068a`.
- Fresh Look Redesign: `76a0c83`.

Re-check the GitHub default branches before making implementation decisions. A shared Graphify workspace means the repositories can be queried together; it does **not** mean they share runtime code or are connected in production.

## 2. TradingAgents

**Observed stack and responsibilities**
- Python agent workflow with graph orchestration in `tradingagents/graph/setup.py`.
- Analyst implementations include `tradingagents/agents/analysts/fundamentals_analyst.py` and `tradingagents/agents/analysts/market_analyst.py`.
- Tool functions in `tradingagents/agents/tools.py` include technical indicator, market snapshot, and macro-indicator retrieval.
- Symbol normalization and safety helpers live in `tradingagents/dataflows/symbols.py`; ticker path validation is implemented by `safe_ticker_component()`.
- Yahoo-based company profile retrieval is implemented by `get_company_profile()` in `tradingagents/dataflows/vendors/yahoo/fundamentals.py`.
- `docker-compose.yml` defines a `tradingagents` service, an `.env` file, and a persistent data volume.

**Integration implication (proposed, not existing)**
Treat TradingAgents as an optional research/analysis worker behind a narrow, documented interface. Do not assume an HTTP API exists; verify how it is launched and invoked before designing a connector.

## 3. Binany AI Trading Bot

**Observed stack and responsibilities**
- TypeScript with Express; `server.ts` constructs the app and binds to `0.0.0.0:3000`.
- `package.json` scripts include `dev: tsx watch server.ts`, `build: tsc --noEmit`, and `start: tsx server.ts`.
- `makeSignal()` in `server.ts` scores strategies including EMA/RSI, Bollinger midline, latest-candle price action, and recent support/resistance; it returns `CALL`, `PUT`, or `WAIT` with a confidence field and reasons.
- `fetchBinanceCandles()` requests Binance klines for symbols it treats as standard assets.
- Important data-quality behavior: synthetic/OTC-like symbols use `generateSimulatedCandles()`; Binance errors/non-success responses also fall back to simulated candles. These synthetic values are generated randomly and are not exchange observations.
- `RiskState` tracks a day, P&L, trade count, and last-trade timestamp in process memory.
- Settings read from environment variables include `MAX_STAKE_USD`, `DAILY_LOSS_LIMIT_USD`, `MAX_TRADES_PER_DAY`, `COOLDOWN_SECONDS`, and `PAPER_MODE`; paper mode defaults to true unless `PAPER_MODE=false`.

**Limitations**
- In-memory risk state is not durable across process restarts.
- The indexed code snapshot alone does not establish that the risk settings are persisted, externally monitored, or sufficient for production.
- Exact HTTP route paths were not confirmed from the retrieved Graphify excerpts; inspect the current `server.ts` before configuring a client. Do not invent endpoints.

## 4. Fresh Look Redesign

**Observed stack and responsibilities**
- TypeScript web app with a server entry at `src/server.ts` and generated route tree at `src/routeTree.gen.ts`.
- UI components live under `src/components/ui/`, including card, form, alert, checkbox, sheet, and navigation components.
- `src/lib/trading.ts` defines trading-related types and a `fetchLiveCandles()` helper.
- `fetchLiveCandles()` requests Binance klines from `data-api.binance.vision`, then `api.binance.com`; it returns candle data if at least 30 rows are available and otherwise ultimately throws `Live feed unavailable`.
- `src/lib/ai-signal.functions.ts` defines `analyzeWithAi` using a server function.

**Integration implication (proposed, not existing)**
Use the web app as a presentation client only after confirming how its server functions and trading utilities are wired. The indexed evidence does not prove it currently calls the Binany Express API.

## 5. Cross-repository relationship map

| Concern | TradingAgents | Binany AI Trading Bot | Fresh Look Redesign |
|---|---|---|---|
| Primary role | Multi-agent research/analysis | Rule-scored signals and Express server | Web UI and trading utilities |
| Language | Python | TypeScript | TypeScript |
| Market data evidence | Multiple dataflow/vendor tools | Binance klines plus simulated fallback | Binance klines via two hosts |
| Signal/analysis evidence | Analyst graph and tools | `makeSignal()` | `makeSignal()` and `analyzeWithAi` symbols are indexed; verify current implementation and call sites |
| Durable runtime integration confirmed? | No | No | No |

No direct import, network call, or deployment link between all three repositories was established by this review. Similar symbol names or shared Graphify indexing are not proof of integration.

## 6. Recommended logical architecture (proposal)

```text
Fresh Look web UI
       |
       | authenticated, versioned API (to be implemented/verified)
       v
Binany Express service :3000
       |                       |
       | market candles        | optional research job/interface
       v                       v
Data validation          TradingAgents worker
       |
       v
Signal + provenance + risk checks
       |
       +--> dashboard / Telegram alerts
       +--> paper-trade ledger
```

This is a target design, not a diagram of the current running system. Keep live execution out of scope unless it is separately designed, explicitly authorized, and protected by independent safeguards.

## 7. Contract and reliability requirements

1. **Data provenance:** Every candle batch and signal should carry source, retrieval timestamp, symbol, timeframe, and a live/simulated flag.
2. **No silent fallback:** Change simulated-candle fallback to an explicit degraded-data state or return a clearly marked simulated result. Do not send simulated signals as live alerts.
3. **Schema validation:** Validate symbol, timeframe, candle count, finite OHLC values, monotonic timestamps, and sensible OHLC relationships.
4. **Deduplication:** Assign stable event IDs from source + symbol + timeframe + candle timestamp + strategy/version. Do not deduplicate solely by a random signal ID.
5. **Risk gates:** Default to paper-only; enforce stake, daily loss, trade count, cooldown, and stale-data limits server-side.
6. **API safety:** Confirm exact routes and request/response schemas from current code. Add authentication, rate limits, request limits, and explicit error responses before exposing an API publicly.
7. **Secrets:** Keep API keys and Telegram bot tokens in environment/secret storage; never commit them.
8. **Observability:** Log source status, validation failures, fallback state, signal IDs, risk-gate decisions, and alert-delivery failures without logging secrets.
9. **Tests:** Cover stale/missing data, simulated fallback, duplicate signals, daily reset, cooldown, and failed notification delivery.
