# Graphify Cross-Repository

A documentation hub for exploring and coordinating three projects indexed in Graphify:

- [TradingAgents](https://github.com/shantocrockerie-cloud/TradingAgents) — Python multi-agent trading research and analysis.
- [Binany AI Trading Bot](https://github.com/shantocrockerie-cloud/binany-ai-trading-bot) — TypeScript/Express signal-generation MVP.
- [Fresh Look Redesign](https://github.com/shantocrockerie-cloud/fresh-look-redesign) — TypeScript web application with trading UI and AI signal-analysis code.

## Purpose

This repository records **verified codebase observations** separately from **proposed integrations**. It is a documentation and planning repository; it does not itself run a trading service, place trades, or establish that the three projects are already connected.

## Start here

- [Cross-repository architecture](docs/architecture.md)
- [Integration roadmap and safety gates](docs/integration-roadmap.md)

## Current scope and safety

- Analysis, signal display, alerts, and paper-trading planning only.
- No live broker/exchange execution is specified or enabled by this documentation.
- Validate live market data before treating a signal as real. The Binany bot's `fetchBinanceCandles()` falls back to randomly generated simulated candles on certain failures and for synthetic/OTC-like symbols. Such output must be labelled **SIMULATED**, never presented as verified live-market data.
- Do not commit API keys, Telegram tokens, credentials, personal data, or populated `.env` files.
- Verify current source before relying on these notes; Graphify indexes can lag behind repository changes.

## Source repositories

| Project | Main verified files / symbols |
|---|---|
| TradingAgents | `tradingagents/graph/setup.py`, `tradingagents/agents/tools.py`, `tradingagents/agents/analysts/fundamentals_analyst.py`, `tradingagents/dataflows/symbols.py`, `docker-compose.yml` |
| Binany AI Trading Bot | `server.ts`, `package.json`, `tsconfig.json`; `makeSignal()`, `fetchBinanceCandles()`, `generateSimulatedCandles()`, `RiskState` |
| Fresh Look Redesign | `src/lib/trading.ts`, `src/lib/ai-signal.functions.ts`, `src/server.ts`, `src/routeTree.gen.ts`, `src/components/ui/*` |

## Status

Initial notes are based on Graphify's indexed snapshots. They are not a substitute for checking current default branches, runtime configuration, or end-to-end tests.
