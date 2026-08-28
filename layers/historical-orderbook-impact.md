# Historical Orderbook Impact

## Purpose

This layer evaluates whether current Paribu orderbook behavior resembles
historical orderbook behavior that was followed by meaningful price movement.

This is an evidence layer.

It does not:

- make BUY/SELL decisions
- replace P5 V2
- replace Event Detector
- replace Lifecycle
- replace Global Evidence
- override Decision Engine

---

## Core Principle

Do not evaluate an order only by its absolute size.

Evaluate:

order appears
→ persists or evolves
→ price approaches/interacts
→ order migrates, resolves or remains
→ price responds
→ similar historical behavior is identified

The historical relationship between order behavior and subsequent price response
is more important than the absolute order size.

---

## Historical Sample

Use the complete available orderbook history.

Do not use a fixed number of snapshots.

Use all available historical snapshots that are relevant to the current market regime.

Repeated observations of the same evolving order should be treated as one
order lifecycle rather than independent events.

---

## Price Distance

Evaluate current liquidity in separate price-distance bands:

- 0–0.5%
- 0.5–1%
- 1–2%
- 2–5%

Do not assume that the closest band is automatically the most important.

More distant liquidity becomes relevant as price approaches it.

---

## Buy-Side Analysis

Evaluate:

- persistence
- upward price migration
- downward migration
- reappearance
- resolution
- size changes
- whether the bid remains during price declines
- whether new bids appear at higher prices
- subsequent price response

A large bid is not automatically bullish.

---

## Sell-Side Analysis

Evaluate:

- persistence
- upward migration
- downward migration
- reappearance
- resolution
- size changes
- whether asks disappear during price rises
- whether asks repeatedly renew
- whether asks follow price upward
- subsequent price response

A large ask is not automatically bearish.

---

## Historical Similarity

Compare current orderbook structures with historical examples using:

- side
- price-distance band
- relative size
- persistence
- migration
- reappearance
- resolution
- surrounding bid/ask balance
- market participation
- subsequent price response

Prefer multiple comparable historical examples.

Do not treat one historical example as reliable evidence.

---

## Volume Context

Use matching units.

For coin quantity:

order_quantity / 24h_coin_volume

For TL value:

order_TL_value / 24h_TL_turnover

Never mix these units.

Absolute TL value alone is not sufficient evidence.

A large order relative to turnover is still not directional evidence unless
its behavior and subsequent price response support that interpretation.

---

## Historical Price Response

For comparable historical orderbook events evaluate subsequent price movement.

Classify the observed response as:

- UP
- DOWN
- NEUTRAL

Where possible evaluate:

- short-term response
- persistence of the response
- magnitude of the response

Historical behavior is evidence, not a guaranteed forecast.

---

## Current Impact Assessment

For the current orderbook determine:

- whether similar orders historically mattered
- historical directional response
- persistence pattern
- migration pattern
- resolution pattern
- historical sample count
- similarity between current and historical behavior

The key question is:

"Does the current orderbook resemble historical orderbook behavior that
previously produced meaningful price movement?"

---

## Evidence Strength

### STRONG

Multiple comparable historical observations show a consistent relationship
between orderbook behavior and subsequent price movement.

### MEDIUM

Comparable historical observations exist, but the relationship is mixed
or incomplete.

### LOW

Evidence exists but is weak or inconsistent.

### INSUFFICIENT_DATA

There are not enough comparable historical observations.

Never manufacture statistical confidence.

---

## Required Output

For each coin provide:

- historical_buy_impact
- historical_sell_impact
- persistence_evidence
- migration_evidence
- resolution_evidence
- price_response_evidence
- historical_sample_count
- confidence

Where available also provide:

- current_relevant_buy_levels
- current_relevant_sell_levels
- current_buy_tl_value
- current_sell_tl_value
- relative_volume_context
- historical_directional_bias

---

## Relationship With Other Layers

Historical Orderbook Impact is evaluated alongside:

- P5 V2
- Event Detector
- Lifecycle
- Global Evidence
- Paribu market evidence

It must never override another evidence layer by itself.

Decision Engine remains responsible for the final decision-support output.

---

## Important Constraint

Do not say:

"This order will move price."

Use evidence-based language:

"Similar historical orderbook behavior was followed by upward price movement."

or:

"Historical evidence does not show a reliable directional response."

The layer provides historical evidence, not deterministic prediction.
