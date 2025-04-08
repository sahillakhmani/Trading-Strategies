# Implied Volatility Pair Trading Strategy: BankNifty vs Nifty

This repository implements an implied volatility (IV) pair trading strategy between BankNifty and Nifty options. The strategy aims to exploit mean-reversion behavior in the IV spread between these two indices.

---

## Strategy Overview

The strategy operates on **minute-level implied volatility (IV) data** of BankNifty and Nifty options.

### IV Spread
- **IV Spread = IV(BankNifty) - IV(Nifty)**
- We expect the spread to be mean-reverting over time.

### Z-Score Calculation
- Rolling mean and standard deviation of the IV spread are computed over a lookback window.
- Z-Score is calculated as:

  ```
  z_score = (iv_spread - mean) / std
  ```

### Signal Logic
- **Entry**:
  - Long spread when z-score < − *z_entry* (I have kept it 0)
  - Short spread when z-score > + *z_entry* (I have kept it 1.3)
- **Exit**:
  - When z-score crosses zero, or goes below *z_exit* threshold, or after max holding period
- **Constraints**:
  - Trades only during market hours (9:15 AM to 3:30 PM)
  - Minimum(30) and maximum holding(375*5) times are enforced (in market minutes)

### PnL Calculation
Profit and loss is computed based on the **change in IV spread**, scaled by time to expiry (TTE):

```python
pnl = signal.shift(1) * spread_change * (tte ** tte_exponent)
```

#### Why Spread Change?
- Trading profits come from **how the spread moves after entering a position**, not from its absolute level.
- A long position profits if the spread increases, and a short position profits if the spread decreases.

#### Why TTE Adjustment?
- Shorter-dated options are more sensitive to IV changes.
- The strategy scales PnL by `TTE ** tte_exponent` to account for this sensitivity.

---

## Backtest Metrics
The following performance metrics are computed:

- **Total Return**: Final cumulative PnL
- **Annualized Return**: Total Return scaled to 252 trading days
- **Annualized Volatility**: Based on standard deviation of daily PnL
- **Sharpe Ratio**: Risk-adjusted return
- **Max Drawdown**: Worst drop from peak cumulative PnL
- **Number of Trades**: Total trades executed based on entry/exit logic

---

## Visualization
Three key plots are produced:
1. **IV Spread with Trade Entries**
2. **Z-Score and Threshold Bands**
3. **Cumulative PnL Over Time**

---

## Results Snapshot (Example)
```
Total Return: 67.65
Annualized Return: 35.35%
Annualized Volatility: 10.07%
Sharpe Ratio: 3.51
Max Drawdown: 3.25
Number of Trades: 646
```

---

## Remarks
- The strategy uses a pure statistical arbitrage approach and does not incorporate transaction costs.
- It is sensitive to the choice of `lookback`, `z_entry`, and `tte_exponent`, which should be carefully tuned.
- TTE-adjusted IV spread is a useful signal proxy for IV pair trades between closely related indices.

---

Feel free to modify signal thresholds and lookback windows to adapt the strategy for different volatility regimes or instruments.

