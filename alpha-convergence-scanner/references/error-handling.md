# Error Handling & Fallback Matrix

## Philosophy

Never let a single failure kill the pipeline. Every gate has a fallback. The scanner degrades gracefully — lower confidence, not zero output.

## Error Scenarios

### Phase 1 — Market Quality Gate

| Error | Retry? | Fallback | Confidence Impact |
|-------|--------|----------|-------------------|
| API timeout (30s) | Yes, once | Widen filters: $25M mcap, $1M liquidity | -15 points |
| API error 5xx | Yes, once | Widen filters: $25M mcap, $1M liquidity | -15 points |
| API rate limit (429) | No | Surface rate-limit message to user. Do NOT retry — shared key may be throttled for all users | Pipeline halts |
| Empty results ($50M threshold) | N/A | Retry at $25M threshold | -15 points |
| Empty results ($25M threshold) | N/A | Retry at $10M threshold + warning | -25 points |
| Empty results ($10M threshold) | N/A | Return: "No qualifying tokens. Market unfavorable. Check in 1-4h." | Pipeline halts |
| JSON parse failure | No | Skip that token, continue with others | None (token dropped) |

### Phase 2 — Smart Money Conviction Gate

| Error | Retry? | Fallback | Confidence Impact |
|-------|--------|----------|-------------------|
| API timeout (30s) | Yes, once | Mark token as "no signal data" | -20 points |
| API error 5xx | Yes, once | Mark token as "no signal data" | -20 points |
| API rate limit (429) | No | Mark ALL remaining tokens as "no signal data" | -20 points each |
| Empty results (no signals for token) | N/A | Mark as "no signal — unconfirmed" | -10 points |
| JSON parse failure | No | Skip signal entry, continue parsing others | None |

### Phase 3 — Security Clearance Gate

| Error | Retry? | Fallback | Confidence Impact |
|-------|--------|----------|-------------------|
| API timeout (30s) | Yes, once | Mark as "PASS with caveat — scan unavailable" | -10 points |
| API error 5xx | Yes, once | Mark as "PASS with caveat — scan unavailable" | -10 points |
| API rate limit (429) | No | Mark remaining tokens with caveat | -10 points each |
| Empty result (no flags) | N/A | PASS — treat as clean | None |
| Result with warning flags | N/A | Surface warnings in trade card | -5 per warning |

### Phase 4 — Scoring

| Error | Fallback |
|-------|----------|
| Can't compute score (missing data) | Base score 40, mark "LOW CONFIDENCE" |
| Only 1 gate has data | Base score 25, mark "SINGLE SOURCE — HIGH RISK" |
| Only 2 gates have data | Average the 2 available scores, mark "PARTIAL DATA" |

## Retry Logic

- Maximum 1 retry per API call
- 2-second delay between retry and original call
- All retries use identical parameters
- If retry succeeds, use the retry response (not the original error)

## Timeout Budget

- Total pipeline timeout: 120 seconds
- Phase 1: 30s max
- Phase 2: 60s max (multiple sequential calls)
- Phase 3: 30s max (multiple sequential calls)
- If any phase exceeds budget, skip remaining tokens in that phase and proceed with what's available.

## Cross-Phase Dependencies

- Phase 2 depends on Phase 1 output (token list)
- Phase 3 depends on Phase 2 output (filtered token list)
- Phase 4 depends on all 3 phases

If Phase 1 returns zero tokens, skip Phases 2-4 entirely.
