Alpha Convergence Scanner
Multi-source trading signal scanner for onchainOS. Requires convergence from 3 independent data sources before surfacing any trade. Built for the OKX Agentic Trading Contest 2026.

Problem
Single-signal bots have high false-positive rates. This scanner eliminates noise by requiring agreement from 3 gates: Market Quality, Smart Money Conviction, and Security Clearance. Only tokens clearing all 3 get scored and surfaced.

Gates
Gate 1 — Market Quality uses onchainos token hot-tokens to filter by market cap (≥$50M), liquidity (≥$2M), and volume (≥$100K). Kills microcaps and dead tokens.

Gate 2 — Smart Money Conviction uses onchainos signal list to check wallet count, buy volume, and soldRatio. Tokens where smart money sold 70%+ are rejected outright.

Gate 3 — Security Clearance uses onchainos security token-scan to catch honeypots, blacklists, freeze functions, and concentrated bundles.

Scoring
Score = (Market × 0.35) + (Smart Money × 0.40) + (Security × 0.25). Smart money weighted highest because it's hardest to fake. 80+ = high conviction, 60-79 = moderate, 40-59 = low, below 40 = no trade.

Key Edges
soldRatio filter — proven in live trading to separate real accumulation from dump traps. Avoided a -14.3% loss while capturing a +2.3% gain.

Quality floor — $50M mcap minimum eliminates 99% of rugs before the pipeline starts.

Graceful degradation — API failures reduce confidence rather than halting the pipeline. Full error matrix in references/error-handling.md.

Trigger Phrases
"Scan for entries", "find trades", "alpha scan", "smart money picks", "what should I buy", "conviction plays", "signal scan."

Files
SKILL.md — Full execution pipeline and skill definition
references/convergence-logic.md — Detailed scoring tables and formulas
references/error-handling.md — Error recovery matrix and timeout budgets
License
MIT
