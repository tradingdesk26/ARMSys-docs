# 🛡️ ARMSys: Automated Risk Management System

ARMSys is an oracle-free execution guard for Uniswap v4 designed to protect Liquidity Providers (LPs) from Loss Versus Rebalancing (LVR). [cite_start]Drawing on 15 years of quantitative experience in TradFi (primarily options trading), ARMSys brings proven, classic risk management approaches on-chain.

By operating directly at the intra-block level, the protocol utilizes an adapted Parkinson volatility model and Z-score risk management to dynamically adjust pool fees or act as a circuit breaker during extreme market stress.

## 📖 Table of Contents
- [Core Technology](#core-technology)
- [Dynamic Fee Mechanism](#dynamic-fee-mechanism)
- [System Architecture](#system-architecture)
- [Security & Audits](#security--audits)

---

## 🧠 Core Technology

Standard AMMs suffer because they rely on discrete, lagging price updates, allowing arbitrageurs to extract value from LPs during volatile swings. ARMSys solves this by tracking price acceleration continuously.

### Moving Beyond Brownian Motion
The classic Parkinson volatility formula relies on the assumption of a standard Brownian motion (continuous random walk) and uses the constant $4\ln2$. However, crypto asset prices (like ETH) do not follow a strict Brownian distribution; they exhibit "fat tails" and impulse movements.

Drawing inspiration from Michael Parkinson's later research on stable probability distributions with a tail parameter < 2 (Pareto-Levy distributions), ARMSys utilizes a recalculated constant tailored specifically for the crypto market's non-standard distribution.

The core mathematical engine analyzes historical data across 4 micro-timeframes (5, 15, 30, and 60 minutes). The core calculation for extreme value variance is defined as:

$$\mathbf{E}[HL^2] = \frac{1}{N} \sum_{t=1}^{N} \left(\ln \frac{H_t}{L_t} \right)^2$$

> **Note:** While the current MVP uses this adapted Parkinson model, the next on-chain iteration will lean heavily into Z-score modeling to ensure maximum gas optimization during intra-block execution.

---

## ⚙️ Dynamic Fee Mechanism

ARMSys translates statistical volatility thresholds into actionable on-chain logic via Uniswap v4 hooks. The system categorizes market conditions into distinct risk modes based on specific ratio thresholds, dynamically adjusting the LP fee.

The protocol is designed to maximize time spent in the **ELEVATED** mode. This is the optimal state where the market exhibits higher-than-normal volatility—allowing LPs to collect double fees—but has not yet reached crash levels that cause severe impermanent loss.

* 🟢 **Normal Mode:**
  * Base condition for standard trading.
  * **Fee:** 0.3% (`3000`)

* 🟡 **ELEVATED Mode:**
  * **Trigger:** `RATIO_ELEVATED = 1.004764e18`
  * **Action:** The protocol dynamically increases the pool fee.
  * **Fee:** 0.6% (`6000`)

* 🟠 **OBSERVATION (HIGH) Mode:**
  * **Trigger:** `RATIO_HIGH = 1.007588e18`
  * **Action:** High-risk zone indicating potential toxic flow. Fees are maximized to heavily penalize arbitrageurs.
  * **Fee:** 1.0% (`10000`)

* 🔴 **EXTREME Mode:**
  * **Trigger:** `RATIO_EXTREME = 1.012061e18`
  * **Action:** Extreme price shock. The hook acts as a circuit breaker, fully blocking toxic execution to protect LP capital until volatility normalizes.

-
