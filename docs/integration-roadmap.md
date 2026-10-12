# Integration Roadmap and Safety Gates

This is a proposed implementation plan, not a statement that these integrations already exist.

## Phase 1 — Establish a trustworthy baseline

- [ ] Re-index all three source repositories after the latest commits.
- [ ] Verify the exact Express routes and schemas in Binany `server.ts`.
- [ ] Verify Fresh Look server-function invocation paths and any environment variables.
- [ ] Verify the supported TradingAgents CLI/entrypoint and how analysis results are returned.
- [ ] Add a health/status check and a visible build/commit identifier to each service where appropriate.

**Exit gate:** current source and startup steps are documented; no endpoint is guessed.

## Phase 2 — Make market-data provenance explicit

- [ ] Mark every data response as `live`, `simulated`, or `unavailable`.
- [ ] Prevent simulated fallback candles from being presented as real market prices.
- [ ] Validate OHLC values, timestamps, symbol/timeframe mappings, and minimum history.
- [ ] Reject stale data and surface provider errors clearly.

**Exit gate:** a provider outage cannot silently generate a live-looking trading alert.

## Phase 3 — Define a versioned signal contract

Suggested contract fields:

- `schema_version`
- `signal_id` and stable `dedupe_key`
- `symbol`, `market`, `timeframe`, `generated_at`
- `direction` (`CALL`, `PUT`, or `WAIT`)
- `confidence`, `reasons`, and indicator summary
- `data_source`, `data_mode`, `data_timestamp`
- `strategy_version`, `risk_gate_status`

Do not treat the example field list as an existing API contract until implemented and tested.

## Phase 4 — Connect dashboard and notifications

- [ ] Have the UI consume a verified, versioned API rather than duplicating signal-generation logic.
- [ ] Add Telegram alerts only after schema validation and risk gates.
- [ ] Deduplicate repeated alerts using the stable key.
- [ ] Include symbol, timeframe, direction, confidence, data mode, source timestamp, and a clear paper-only label.
- [ ] Retry transient delivery failures with bounded retries and an idempotency strategy.

## Phase 5 — Paper-trading and risk tests

- [ ] Keep real-money execution disabled.
- [ ] Enforce maximum stake, daily loss limit, trade count, cooldown, and data-freshness gates.
- [ ] Persist the paper ledger and risk state if restart continuity is required.
- [ ] Test daily rollover, duplicate events, invalid candles, provider outage, and notification failure.
- [ ] Compare results against a baseline; do not assume confidence scores equal calibrated win probabilities.

## Non-negotiable release gates

- Live and simulated market data are visibly distinct.
- No unverified API endpoint is used.
- No secrets are stored in Git.
- Risk checks run server-side and fail closed for missing/invalid inputs.
- Paper-trading and alerts are the only enabled actions.
- Logs and dashboard expose failures instead of hiding them.
