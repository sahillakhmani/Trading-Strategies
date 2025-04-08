1. __init__(self, lookback=375*3, z_entry=1.5, z_exit=0.5, tte_exponent=0.7, min_holding=30, max_holding=375*5, trading_hours=(time(9,15), time(15,30)))
Purpose: Initializes the pair trading strategy with configurable parameters.
Parameters:
lookback: Window size (in minutes) for calculating rolling mean and standard deviation of the IV spread.
z_entry: Z-score threshold to enter a trade (e.g., 1.5 means enter when z-score > 1.5 or < -1.5).
z_exit: Z-score threshold to exit a trade (e.g., 0.5 means exit when z-score falls below 0.5 in magnitude).
tte_exponent: Exponent applied to time-to-expiry (tte) in profit/loss calculation to adjust for option decay.
min_holding: Minimum holding period (in minutes) before exiting a trade.
max_holding: Maximum holding period (in minutes) before forcing an exit.
trading_hours: Tuple of start and end times (e.g., 9:15 AM to 3:30 PM) to filter data.
Behavior: Sets up instance variables and initializes data and results as None.

2. load_data(self, filepath)
Purpose: Loads and preprocesses data from a Parquet file.
Parameters:
filepath: Path to the Parquet file containing the data.
Behavior:
Reads the Parquet file into a Pandas DataFrame.
Ensures the index is a DatetimeIndex (converts datetime column if present).
Filters data to trading hours (e.g., 9:15 AM to 3:30 PM).
Forward-fills missing values in banknifty, nifty, and tte columns.
Calculates the IV spread as banknifty - nifty.
Raises an error if the spread contains NA values.

3. calculate_zscores(self)
Purpose: Calculates the z-scores of the IV spread and visualizes them.
Behavior:
Computes rolling mean (spread_mean) and standard deviation (spread_std) of the IV spread using the lookback window.
Calculates z-scores: (iv_spread - spread_mean) / spread_std.
Plots two subplots:
IV spread with its rolling mean.
Z-score with entry (z_entry) and exit (z_exit) thresholds as horizontal lines.
Output: Displays the plots using Matplotlib.

4. generate_signals(self)
Purpose: Generates trading signals based on z-scores and holding period rules.
Behavior:
Initializes signal, entry_time, and holding_period columns in the DataFrame.
Iterates through the data to:
Enter a trade when z_score >= z_entry (short spread, signal = -1) or z_score <= -z_entry (long spread, signal = 1).
Exit a trade when:
holding_period >= min_holding and either:
Z-score drops below z_exit in magnitude.
Z-score crosses zero (mean reversion).
holding_period >= max_holding.
Tracks holding period in market minutes (holding_counter).
Forward-fills signals to maintain positions between entry and exit.

5. calculate_pnl(self)
Purpose: Calculates profit and loss (PnL) based on trading signals.
Behavior:
Computes the change in IV spread (spread_change).
Calculates daily PnL: signal.shift(1) * spread_change * (tte ** tte_exponent).
signal.shift(1): Uses the previous signal to avoid look-ahead bias.
tte ** tte_exponent: Adjusts PnL for time-to-expiry decay.
Computes cumulative PnL (cumulative_pl) as the running sum of daily PnL.

6. backtest(self)
Purpose: Runs the full backtest by calling other methods and computing performance metrics.
Behavior:
Calls calculate_zscores(), generate_signals(), and calculate_pnl().
Calculates:
Total return: Final cumulative PnL.
Annualized return: (total_return / total_days) * 252.
Volatility: Annualized standard deviation of daily PnL.
Sharpe ratio: annualized_return / volatility.
Max drawdown: Maximum peak-to-trough decline in cumulative PnL.
Number of trades: Count of trade entries/exits.
Returns a dictionary (results) with these metrics.

7. plot_results(self)
Purpose: Visualizes the backtest results.
Behavior:
Plots three subplots:
IV spread with trade entries (green for long, red for short).
Z-score with entry and exit thresholds.
Cumulative PnL with a zero line.
Displays the plots using Matplotlib.
