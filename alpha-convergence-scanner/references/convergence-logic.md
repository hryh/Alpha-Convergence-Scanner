# Convergence Logic — Detailed Scoring

## Scoring System (Unified)

Each gate produces a score in its own range. The final convergence score is a simple sum:

```
Convergence Score = Market Score + Smart Money Score + Security Score  (max 100)
```

This is simpler and more transparent than weighted multiplication. Each gate's max is its weight — no separate formula needed.

---

## Market Quality Score (0-35)

Based on Phase 1 results. Higher = more established, more liquid, safer.

| Metric | Threshold | Points |
|--------|-----------|--------|
| Market Cap | ≥ $500M | 12 |
| | $200M-$499M | 9 |
| | $100M-$199M | 6 |
| | $50M-$99M | 3 |
| | < $50M (fallback) | 1 |
| Liquidity | ≥ $10M | 8 |
| | $5M-$9.9M | 6 |
| | $2M-$4.9M | 4 |
| | $1M-$1.9M (fallback) | 2 |
| Holders | ≥ 100K | 7 |
| | 50K-99K | 5 |
| | 10K-49K | 3 |
| | < 10K | 1 |
| 24h Change | +2% to +15% | 5 |
| | 0% to +2% | 3 |
| | -5% to 0% (dip buy) | 3 |
| | < -5% or > +15% | 0 |
| Volume 24h | ≥ $1M | 3 |
| | $500K-$999K | 2 |
| | $100K-$499K | 1 |

---

## Smart Money Conviction Score (0-40)

| Metric | Threshold | Points |
|--------|-----------|--------|
| Wallet Count | ≥ 8 | 20 |
| | 5-7 | 14 |
| | 3-4 | 8 |
| | 1-2 | 4 |
| Buy Volume (USD) | ≥ $5,000 | +8 |
| | $1,000-$4,999 | +5 |
| | $500-$999 | +3 |
| | < $500 | +1 |
| soldRatio Bonus | < 30% | +12 |
| soldRatio Neutral | 30-50% | 0 |
| soldRatio Penalty | 50-70% | -12 |
| soldRatio Reject | > 70% | **REJECT** |

---

## Security Score (0-25)

- Base: 20 points (clean scan — no flags)
- Bundle concentration < 20%: +5 (total 25)
- Bundle concentration 20-40%: +0 (total 20)
- Bundle concentration > 40%: -10 (total 10)
- Any of: honeypot, blacklist, freeze, tax > 10%: **score = 0, token REJECTED**

---

## Entry Parameter Formulas (Corrected)

### Position Size
```
suggestedSizePercent = min(25, max(5, convergenceScore / 4))
```
- Score 90: min(25, 22.5) = 22.5%
- Score 76: min(25, 19.0) = 19.0%
- Score 60: min(25, 15.0) = 15.0%
- Score 50: min(25, 12.5) = 12.5%

### Stop Loss
```
stopLossPct = max(0.02, (100 - convergenceScore) / 1000)
stopLoss = currentPrice × (1 - stopLossPct)
```
- Score 90: max(0.02, 0.010) = 2.0% → verified ✓
- Score 76: max(0.02, 0.024) = 2.4% → verified ✓
- Score 60: max(0.02, 0.040) = 4.0% → verified ✓
- Score 50: max(0.02, 0.050) = 5.0% → verified ✓

### Take Profit
```
takeProfitPct = max(0.05, min(0.20, convergenceScore / 500))
takeProfit = currentPrice × (1 + takeProfitPct)
```
- Score 90: max(0.05, min(0.20, 0.18)) = 18.0% → verified ✓
- Score 76: max(0.05, min(0.20, 0.152)) = 15.2% → verified ✓
- Score 60: max(0.05, min(0.20, 0.12)) = 12.0% → verified ✓
- Score 50: max(0.05, min(0.20, 0.10)) = 10.0% → verified ✓

---

## Decision Matrix

| Score | Label | Size | SL | TP | Confidence |
|-------|-------|------|-----|-----|------------|
| 90 | HIGH | 22.5% | 2.0% | 18.0% | Strong buy |
| 80 | HIGH | 20.0% | 2.0% | 16.0% | Strong buy |
| 76 | MOD-HIGH | 19.0% | 2.4% | 15.2% | Decent |
| 70 | MODERATE | 17.5% | 3.0% | 14.0% | Decent |
| 60 | MODERATE | 15.0% | 4.0% | 12.0% | OK |
| 50 | LOW | 12.5% | 5.0% | 10.0% | Marginal |
| 40 | LOW | 10.0% | 6.0% | 8.0% | Risky |
| < 40 | REJECT | 0% | — | — | Skip |

---

## Real-World Example: Our RAY Trade (May 10, 2026)

```
Token: RAY (Raydium) | 4k3Dyjzvzp8eMZWUXbBCjEvwSkkk59S5iCNLY3QrkX6R

Gate 1 — Market Quality:
  Mcap $484M → 9pts | Liq $12M → 8pts | Holders 296K → 7pts
  +2.6% 24h → 5pts | Vol $2.8M → 3pts
  Market Score: 32/35

Gate 2 — Smart Money:
  5 wallets → 14pts | Buy vol $2.3K → 5pts | soldRatio 35% → 0pts
  Smart Money Score: 19/40

Gate 3 — Security:
  Clean scan → 20pts | Bundle <1% → +5pts
  Security Score: 25/25

CONVERGENCE SCORE: 32 + 19 + 25 = 76/100 → MODERATE-HIGH

Entry params:
  Size: min(25, 76/4) = 19.0% of portfolio ($97 on $513)
  Stop: $0.873 × (1 - 0.024) = $0.852
  Take: $0.873 × (1 + 0.152) = $1.006

Entry: $0.872 (actual fill)
Current: $0.892 (+2.3% unrealized)

What the skill would have done differently:
  - Would have scored BMNTP and auto-rejected at Gate 2 (soldRatio 100%)
  - Would have scored JTO at ~55-60 (MODERATE) with lower size (12.5%)
  - Would have ranked RAY #1 with highest conviction
  ✓ All 3 recommendations match optimal trading decisions
```
