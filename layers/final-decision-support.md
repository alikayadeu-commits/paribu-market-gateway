# Final Decision Support

## Purpose

This document defines the final decision-support architecture
for the Paribu market gateway.

The system separates:

1. Local Paribu market interpretation
2. Global market confirmation
3. Final decision support

The GitHub automation is responsible for the local Paribu layer.

Binance and OKX are NOT fetched or interpreted by the GitHub
workflow.

They are evaluated live through the available ChatGPT connectors
when final decision support is requested.

---

## 1. LOCAL PARIBU LAYER

Primary source:

    state/decision.json

The local Paribu decision engine evaluates:

- Paribu market data
- price
- volume
- orderbook pressure
- detected events
- lifecycle state
- reversal persistence
- local decision state

Possible local states include:

    ADD_CONSIDER
    REDUCE_PROFIT
    WAIT
    NONE

The local layer is authoritative for Paribu-specific evidence.

---

## 2. GLOBAL CONFIRMATION LAYER

Global confirmation is obtained live from:

    Binance connector
    OKX connector

The GitHub repository does NOT store a permanent
global_evidence.json contract for this purpose.

The global layer is evaluated at decision time.

For each coin, the global layer determines:

    UP
    DOWN
    FLAT
    UNKNOWN

and whether Binance and OKX agree.

---

## 3. FINAL DECISION SUPPORT

Final decision support combines:

    LOCAL PARIBU EVIDENCE
    +
    BINANCE GLOBAL EVIDENCE
    +
    OKX GLOBAL EVIDENCE

The final result must be one of:

    ADD_CONSIDER
    REDUCE_PROFIT
    WAIT
    NO_ACTION

---

## 4. DECISION RULES

### ADD_CONSIDER

Return ADD_CONSIDER only when:

- Paribu local evidence is directionally positive
- Binance confirms the positive direction
- OKX confirms the positive direction
- there is no material data-health problem
- the evidence is sufficiently aligned

A strong local reversal confirmation is preferred.

A local WAIT state must NOT be upgraded to ADD_CONSIDER
solely because Binance and OKX are positive.

---

### REDUCE_PROFIT

Return REDUCE_PROFIT only when:

- Paribu local evidence is directionally negative
- Binance confirms the negative direction
- OKX confirms the negative direction
- there is no material data-health problem
- the evidence is sufficiently aligned

A strong local reversal confirmation is preferred.

A local WAIT state must NOT be upgraded to REDUCE_PROFIT
solely because Binance and OKX are negative.

---

### WAIT

Return WAIT when:

- local evidence exists but confirmation is incomplete
- Binance and OKX disagree
- local and global evidence conflict
- the reversal is not sufficiently persistent
- global evidence exists but local Paribu confirmation is weak

WAIT means:

    Do not act yet.
    Continue observing.

---

### NO_ACTION

Return NO_ACTION when:

- Paribu data is stale or invalid
- required global evidence cannot be obtained
- the market state is not interpretable
- there is insufficient evidence to support a directional decision

NO_ACTION is a data-safety state.

---

## 5. DATA SAFETY

The final decision must never compensate for missing data
by guessing.

If a required source is unavailable:

    do not infer
    do not fabricate
    do not extrapolate

Use:

    WAIT

or:

    NO_ACTION

depending on whether the missing data represents
uncertainty or a data-health failure.

---

## 6. SEPARATION OF RESPONSIBILITIES

### GitHub Actions

Responsible for:

- fetching/storing Paribu market data
- detecting local events
- maintaining lifecycle state
- maintaining local decision state

### Cloudflare Worker

Responsible for:

- scheduled triggering
- Paribu API gateway
- reliable market-data access

### ChatGPT connectors

Responsible for:

- live Binance market evidence
- live OKX market evidence

### Final Decision Support

Responsible for:

- combining local Paribu evidence
- combining Binance evidence
- combining OKX evidence
- producing the final decision-support result

---

## 7. IMPORTANT ARCHITECTURAL RULE

The repository must NOT attempt to call Binance or OKX
from the Paribu GitHub workflow.

The global connector layer remains outside GitHub Actions.

This prevents duplication of responsibilities and keeps:

    Paribu automation
    global market confirmation
    final decision interpretation

as separate layers.

---

## 8. FINAL OUTPUT FORMAT

When final decision support is requested, the output should
contain:

    Coin
    Local Paribu decision
    Local lifecycle/event
    Paribu orderbook pressure
    Binance direction
    OKX direction
    Global consensus
    Final decision
    Confidence
    Reason

Example:

    VIRTUAL
    Local: ADD_CONSIDER
    Binance: UP
    OKX: UP
    Consensus: CONFIRMED
    Final: ADD_CONSIDER
    Confidence: HIGH

The final output is decision support.

It is not an automatic trade execution command.

No order is placed automatically.

---

## 9. CURRENT MODE

Mode:

    DECISION_SUPPORT

Execution:

    MANUAL

Order execution:

    DISABLED

The system may recommend:

    ADD_CONSIDER
    REDUCE_PROFIT

but it must never execute a trade automatically.
