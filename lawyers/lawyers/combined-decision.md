# Combined Decision Layer

## Purpose

Combine:

1. Paribu-local decision state from GitHub
2. Live global market evidence from ChatGPT market connectors

to produce a final evidence-aware decision.

This layer does NOT collect market data itself.

It does NOT write Binance or OKX data into GitHub.

It does NOT create a new trading strategy.

It only combines the existing local Paribu decision with independent global market evidence.

---

# 1. INPUT A — PARIBU / GITHUB

Read the latest Decision Engine output from:

state/decision.json

For each:

- VIRTUAL
- FET
- KITE

use:

- decision
- last_event
- last_lifecycle_state
- price
- volume
- orderbook_pressure
- last_push_key
- reason

The local Paribu decision remains the primary event source.

---

# 2. INPUT B — GLOBAL MARKET

Use live ChatGPT market connectors.

Required independent sources:

- Binance
- OKX

Required instruments:

VIRTUAL/USDT
FET/USDT
KITE/USDT
BTC/USDT

For each coin collect, when available:

- current price
- 24h percentage change
- 24h volume
- timestamp / freshness

For BTC collect:

- current price
- 24h percentage change
- timestamp / freshness

Never use GitHub as a proxy for live Binance or OKX data.

Never invent connector data.

If a connector is unavailable, explicitly mark it unavailable.

---

# 3. GLOBAL DIRECTION

Classify each global asset:

UP:
24h change > +0.25%

DOWN:
24h change < -0.25%

FLAT:
-0.25% to +0.25%

For a coin:

GLOBAL_UP_CONFIRMED:
Binance and OKX both show UP.

GLOBAL_DOWN_CONFIRMED:
Binance and OKX both show DOWN.

GLOBAL_CONFLICT:
Binance and OKX are both available but point in opposite directions.

GLOBAL_PARTIAL:
Only one independent source is available.

GLOBAL_UNKNOWN:
No usable global source is available.

---

# 4. BTC CONTEXT

BTC is contextual evidence.

BTC_UP:
BTC > +0.25%

BTC_DOWN:
BTC < -0.25%

BTC_FLAT:
BTC between -0.25% and +0.25%

BTC does not independently create a coin signal.

BTC must never override strong coin-specific evidence.

---

# 5. LOCAL EVENT CLASSIFICATION

Interpret the existing Decision Engine state.

Positive local event examples:

- BREAKOUT_UP
- MOMENTUM_UP
- REVERSAL_UP
- DEMAND_PRESSURE
- BUY-type decision

Negative local event examples:

- BREAKOUT_DOWN
- MOMENTUM_DOWN
- REVERSAL_DOWN
- SUPPLY_PRESSURE
- REDUCE/SELL-type decision

Neutral:

- NONE
- WAIT
- no meaningful event

Do not reinterpret a neutral event as a trade signal.

---

# 6. COMBINATION RULES

## Case A — Local positive + global positive

If:

LOCAL = positive

and

GLOBAL = UP_CONFIRMED

then:

combined_status = CONFIRMED

action = PASS

confidence = HIGH

provided that both global sources are fresh and usable.

---

## Case B — Local positive + global negative

If:

LOCAL = positive

and

GLOBAL = DOWN_CONFIRMED

then:

combined_status = CONFLICT

action = WAIT

confidence = LOW

Do NOT create a buy push.

---

## Case C — Local negative + global negative

If:

LOCAL = negative

and

GLOBAL = DOWN_CONFIRMED

then:

combined_status = CONFIRMED

action = PASS

confidence = HIGH

This means the local downside event is globally supported.

The layer still does NOT independently decide position size or execution.

---

## Case D — Local negative + global positive

If:

LOCAL = negative

and

GLOBAL = UP_CONFIRMED

then:

combined_status = CONFLICT

action = WAIT

confidence = LOW

Do NOT create a reduce/sell push solely from this conflict.

---

## Case E — Local signal + only one global source

If:

LOCAL = meaningful

and

only one of Binance/OKX is available,

then:

combined_status = PARTIAL_CONFIRMATION

action = WAIT

confidence = MEDIUM

Do not upgrade to HIGH.

---

## Case F — Local signal + global conflict

If:

Binance and OKX materially disagree,

then:

combined_status = GLOBAL_CONFLICT

action = WAIT

confidence = LOW

---

## Case G — No local event

If:

LOCAL = NONE

then:

combined_status = NO_LOCAL_EVENT

action = NO_ACTION

confidence = N/A

Global movement alone must NOT create a trade signal.

---

# 7. FRESHNESS

Global evidence freshness:

0–5 minutes:
FRESH

>5–15 minutes:
STALE_WARNING

>15–30 minutes:
DEGRADED

>30 minutes:
STALE

Only FRESH evidence can participate in HIGH-confidence confirmation.

STALE_WARNING evidence may participate in MEDIUM confidence.

DEGRADED evidence must not produce HIGH confidence.

STALE evidence must not be used as confirmation.

Unavailable data is not bearish or bullish evidence.

---

# 8. VOLUME

Volume is confirmation evidence.

Do not use raw Binance volume versus raw OKX volume as a direct comparison.

Instead evaluate whether the price movement is accompanied by meaningful participation on the respective exchange.

If price movement is strong and participation is supportive:

volume_evidence = SUPPORTIVE

If price movement is weak or participation is unclear:

volume_evidence = NEUTRAL

If the move appears weak relative to available participation data:

volume_evidence = WEAK

Volume alone never creates a signal.

---

# 9. FINAL ACTION BOUNDARY

Allowed actions:

PASS
WAIT
NO_ACTION

This layer must NOT:

- place orders
- calculate position size
- create leverage
- invent entry prices
- invent stop loss levels
- create a push without a valid local event
- override Event Gate
- override State Lifecycle
- create a new local event

The existing Paribu Event Detector and Decision Engine remain authoritative for local events.

---

# 10. OUTPUT FORMAT

For every evaluated coin return:

{
  "coin": "VIRTUAL",
  "local": {
    "decision": "NONE",
    "event": "REVERSAL_DOWN",
    "lifecycle": "REVERSAL_DOWN",
    "price_try": 34.05,
    "volume": 88995.05,
    "orderbook_pressure": "supply_heavy"
  },
  "global": {
    "binance": {
      "available": true,
      "direction": "UP",
      "change_24h_pct": null,
      "freshness": "FRESH"
    },
    "okx": {
      "available": true,
      "direction": "UP",
      "change_24h_pct": null,
      "freshness": "FRESH"
    },
    "btc": {
      "direction": "UP",
      "change_24h_pct": null,
      "freshness": "FRESH"
    },
    "exchange_agreement": true
  },
  "combined": {
    "status": "CONFLICT",
    "action": "WAIT",
    "confidence": "LOW"
  },
  "reason": ""
}

The numerical global values must always come from the live connector query.

Never reuse the example numbers above as live data.

---

# 11. EVIDENCE REASON

The reason must explicitly explain the relationship between local and global evidence.

Example:

"Paribu shows REVERSAL_DOWN while Binance and OKX both show UP. Global evidence therefore contradicts the local downside event. No automatic action."

Another example:

"Paribu shows REVERSAL_UP while Binance and OKX both show UP with fresh data and supportive volume. The local event is independently confirmed."

---

# 12. STRATEGIC PRINCIPLE

The strategy is:

PARIBU EVENT
→ EVENT GATE
→ GLOBAL CROSS-CHECK
→ COMBINED DECISION
→ ACTION / WAIT / NO ACTION

Global market movement alone is never sufficient.

A strong global move cannot manufacture a Paribu event.

A Paribu event cannot be considered strongly confirmed when independent global evidence contradicts it.

The purpose of this layer is:

FEWER FALSE POSITIVES
+
PRESERVE IMPORTANT EVENTS
+
INDEPENDENT CONFIRMATION

The system should prefer waiting over acting when evidence is materially conflicted.

---

# 13. ARCHITECTURE

GitHub provides:

- Paribu market data
- Event Detector state
- Event Tracker / Lifecycle state
- Event Gate state
- Decision Engine state

ChatGPT connectors provide:

- Binance live global market evidence
- OKX live global market evidence

ChatGPT combines these sources using this contract.

Global connector data is NOT copied into GitHub merely to make this layer work.

---

# 14. IMPORTANT

This document is a decision contract.

It is not a GitHub Action.

It does not call APIs.

It does not create live market data.

It does not replace the existing Decision Engine.

It defines how the existing local decision and live global evidence must be combined.
