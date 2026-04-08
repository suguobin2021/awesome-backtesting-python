# Awesome Backtesting Python [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated, opinionated guide to Python backtesting frameworks — with honest comparisons of maintenance status, speed, realism, and learning curve. Updated for 2026.

[![License: CC0-1.0](https://img.shields.io/badge/License-CC0_1.0-lightgrey.svg)](https://creativecommons.org/publicdomain/zero/1.0/)
![Last Verified](https://img.shields.io/badge/Last%20Verified-2026--04-green)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)

**Why this list exists**: r/algotrading's most repeated question is "which backtesting framework should I use?" The answer depends heavily on your use case, yet most comparisons are outdated (Backtrader is still cited everywhere despite stagnating since 2021), incomplete, or written by people who only used one library. This guide is an honest, maintained comparison.

---

## Table of Contents

- [Framework Comparison at a Glance](#framework-comparison-at-a-glance)
- [Framework Deep Dives](#framework-deep-dives)
  - [backtesting.py](#backtestingpy)
  - [VectorBT / VectorBT PRO](#vectorbt--vectorbt-pro)
  - [Zipline-Reloaded](#zipline-reloaded)
  - [bt (Flexible Backtesting for Python)](#bt-flexible-backtesting-for-python)
  - [QuantConnect LEAN](#quantconnect-lean)
  - [Nautilus Trader](#nautilus-trader)
  - [Backtrader (Legacy)](#backtrader-legacy)
- [Critical Pitfalls in Backtesting](#critical-pitfalls-in-backtesting)
- [Performance Metrics Libraries](#performance-metrics-libraries)
- [Data Sources for Backtesting](#data-sources-for-backtesting)
- [Decision Guide: Which Framework to Use?](#decision-guide-which-framework-to-use)
- [Learning Resources](#learning-resources)

---

## Framework Comparison at a Glance

| Framework | Approach | Speed | Realism | Learning Curve | Live Trading | Maintenance (2026) | Best For |
|-----------|----------|-------|---------|---------------|--------------|-------------------|---------|
| `backtesting.py` | Event-driven | Medium | Medium | ★★☆☆☆ | ❌ | 🟡 Slow | Beginners, signal testing |
| `vectorbt` | Vectorized | ★★★★★ | Low-Medium | ★★★☆☆ | ❌ (vectorbtpro yes) | 🟢 Active | Parameter optimization |
| `zipline-reloaded` | Event-driven | Medium | High | ★★★★☆ | ❌ | 🟡 Community | Realistic simulation |
| `bt` | Composition | Medium | Medium | ★★★☆☆ | ❌ | 🟡 Slow | Portfolio strategies |
| `LEAN (QuantConnect)` | Event-driven | Medium | ★★★★★ | ★★★★★ | ✅ | 🟢 Active | Production-grade |
| `nautilus_trader` | Event-driven | ★★★★★ | ★★★★★ | ★★★★★ | ✅ | 🟢 Active | HFT/professional |
| `backtrader` | Event-driven | Low | High | ★★★☆☆ | ✅ | 🔴 Stagnant | Legacy projects only |

**Verdict summary**:
- **Just learning?** → `backtesting.py`
- **Running 10,000+ parameter combinations?** → `vectorbt`
- **Want institutional-grade realism?** → `LEAN (QuantConnect)`
- **Going live?** → `nautilus_trader` or `LEAN`
- **Should I use Backtrader?** → Only if you have existing Backtrader code. New projects should not start with it.

---

## Framework Deep Dives

### backtesting.py

**GitHub**: [kernc/backtesting.py](https://github.com/kernc/backtesting.py) | **Stars**: ~5k | **Last release**: 2023

The most beginner-friendly framework. Clean API, built-in browser-based chart visualization, and a well-designed optimization interface. The event loop is intuitive for anyone with OOP experience.

**Strengths**:
- Minimal boilerplate — a working strategy in under 20 lines
- Built-in `plot()` function with interactive Bokeh charts
- Grid search optimization via `bt.optimize()`

**Weaknesses**:
- No built-in position sizing models (must implement manually)
- Single-asset per run (multi-asset strategies are cumbersome)
- No live trading bridge
- Development has slowed significantly since 2023

**Installation**:
```bash
pip install backtesting
```

**Example — Simple Moving Average Crossover**:
```python
from backtesting import Backtest, Strategy
from backtesting.lib import crossover
import pandas as pd
import yfinance as yf

class SmaCross(Strategy):
    n1 = 50  # Fast SMA period
    n2 = 200  # Slow SMA period

    def init(self):
        close = self.data.Close
        self.sma1 = self.I(lambda x: pd.Series(x).rolling(self.n1).mean(), close)
        self.sma2 = self.I(lambda x: pd.Series(x).rolling(self.n2).mean(), close)

    def next(self):
        if crossover(self.sma1, self.sma2):
            self.buy()
        elif crossover(self.sma2, self.sma1):
            self.sell()

# Download data
data = yf.download('SPY', start='2010-01-01', end='2024-01-01', auto_adjust=True)
data.columns = [c[0] for c in data.columns]  # Fix MultiIndex

bt = Backtest(data, SmaCross, cash=10_000, commission=.002)
stats = bt.run()
print(stats)
bt.plot()

# Optimize n1 and n2
optimized = bt.optimize(n1=range(20, 100, 10), n2=range(100, 300, 20),
                        maximize='Sharpe Ratio', constraint=lambda p: p.n1 < p.n2)
print(optimized._strategy)
```

---

### VectorBT / VectorBT PRO

**GitHub**: [polakowo/vectorbt](https://github.com/polakowo/vectorbt) | **Stars**: ~4k | **Website**: [vectorbt.pro](https://vectorbt.pro)

The fastest Python backtesting library for parameter sweeps. Uses NumPy vectorization and Numba JIT compilation to run thousands of parameter combinations in seconds instead of hours.

**Strengths**:
- Orders of magnitude faster than event-driven approaches for optimization
- Handles multi-asset, multi-strategy simulations natively
- Rich performance analytics with QuantStats integration
- Active development (vectorbtpro is the commercial successor)

**Weaknesses**:
- The vectorized paradigm is harder to reason about (no "loop through bars" mental model)
- Harder to implement realistic fill logic (slippage, partial fills)
- `vectorbtpro` is paid ($199+/year); the open-source `vectorbt` is more limited
- Not suitable for strategies requiring complex state

**Installation**:
```bash
pip install vectorbt  # Open source version
# pip install vectorbtpro  # Commercial version
```

**Example — 10,000 SMA combinations in seconds**:
```python
import vectorbt as vbt
import numpy as np

# Download data
price = vbt.YFData.download('SPY', start='2010-01-01', end='2024-01-01').get('Close')

# Run ALL combinations of fast/slow SMA (100 x 100 = 10,000 combos)
fast_windows = np.arange(10, 110, 1)
slow_windows = np.arange(20, 220, 2)

fast_ma, slow_ma = vbt.MA.run_combs(price, window=fast_windows,
                                      r=2, short_names=['fast', 'slow'])

entries = fast_ma.ma_crossed_above(slow_ma)
exits = fast_ma.ma_crossed_below(slow_ma)

pf = vbt.Portfolio.from_signals(price, entries, exits, freq='1D', init_cash=10_000)

# Find best Sharpe ratio across all combinations
best_idx = pf.sharpe_ratio().idxmax()
print(f"Best combo: fast={best_idx[0]}, slow={best_idx[1]}")
print(f"Best Sharpe: {pf.sharpe_ratio()[best_idx]:.2f}")
```

---

### Zipline-Reloaded

**GitHub**: [stefan-jansen/zipline-reloaded](https://github.com/stefan-jansen/zipline-reloaded) | Community fork of Quantopian's Zipline

The most realistic event-driven simulator available in open source. Designed to prevent look-ahead bias through a rigorous data ingestion pipeline. The original Zipline was the engine behind Quantopian (shut down 2020); `zipline-reloaded` keeps it alive.

**Strengths**:
- Excellent handling of point-in-time data (prevents look-ahead bias)
- Pipeline API for cross-sectional fundamental factor research
- Realistic order fill models

**Weaknesses**:
- Complex setup (requires data bundle ingestion)
- Python 3.8+ compatibility issues crop up occasionally
- Community-maintained; no commercial backing

**Installation**:
```bash
pip install zipline-reloaded
```

**Example — Basic Algorithm**:
```python
from zipline.api import order_target_percent, record, symbol
import zipline

def initialize(context):
    context.asset = symbol('AAPL')

def handle_data(context, data):
    # Access price history without look-ahead bias
    prices = data.history(context.asset, 'price', bar_count=50, frequency='1d')
    sma_20 = prices.mean()
    current_price = data.current(context.asset, 'price')

    if current_price > sma_20:
        order_target_percent(context.asset, 0.95)
    else:
        order_target_percent(context.asset, 0.0)

    record(price=current_price, sma20=sma_20)
```

---

### bt (Flexible Backtesting for Python)

**GitHub**: [pmorissette/bt](https://github.com/pmorissette/bt) | **Stars**: ~2k

A composition-based framework that treats strategies as trees of nodes — each node handles allocation, selection, or rebalancing. Excellent for portfolio-level strategies.

**Strengths**:
- Natural fit for multi-asset portfolio strategies (equal weight, risk parity, etc.)
- Built-in portfolio statistics (risk, drawdown, turnover)
- Clean composition API

**Weaknesses**:
- Less flexible for tick-level or intraday strategies
- Smaller community, slower development

**Installation**:
```bash
pip install bt
```

**Example — Equal Weight Monthly Rebalance**:
```python
import bt
import yfinance as yf

tickers = ['SPY', 'TLT', 'GLD', 'VNQ']
prices = yf.download(tickers, start='2010-01-01', end='2024-01-01',
                     auto_adjust=True)['Close']

strategy = bt.Strategy('EqualWeight', [
    bt.algos.RunMonthly(),           # Rebalance monthly
    bt.algos.SelectAll(),            # Use all available assets
    bt.algos.WeighEqually(),         # Equal weight
    bt.algos.Rebalance()             # Execute rebalance
])

backtest = bt.Backtest(strategy, prices)
result = bt.run(backtest)
result.plot()
result.display()
```

---

### QuantConnect LEAN

**GitHub**: [QuantConnect/Lean](https://github.com/QuantConnect/Lean) | **Stars**: ~10k | **Website**: [quantconnect.com](https://www.quantconnect.com)

The most production-ready open-source backtesting engine. LEAN supports equities, options, futures, forex, and crypto. QuantConnect's cloud platform uses it, but you can also run it locally. Steep learning curve, but the realism and live trading bridges make it worth it for serious projects.

**Strengths**:
- Multi-asset support (equities, options, futures, forex, crypto)
- Realistic fill models with slippage and market impact
- Live trading bridges to Interactive Brokers, Alpaca, etc.
- Cloud-based research environment available

**Weaknesses**:
- Significant learning investment (C#-originated, Python interface is secondary)
- Local setup requires Docker
- Cloud credits for premium data

**Local Installation**:
```bash
# Requires Docker
docker pull quantconnect/lean
# See: https://www.lean.io/docs/v2/lean-cli/getting-started/
```

---

### Nautilus Trader

**GitHub**: [nautechsystems/nautilus_trader](https://github.com/nautechsystems/nautilus_trader) | **Stars**: ~5k

A high-performance trading system with a Rust/Cython core that runs in Python. Designed for live trading from day one, with backtesting as an equal-class citizen. The learning curve is steep, but the performance is unmatched in open source.

**Strengths**:
- Rust core: nanosecond-precision event handling
- Production-ready live trading (Interactive Brokers, Bybit, OKX, etc.)
- Backtesting uses identical code paths as live trading
- Active, professional development team

**Weaknesses**:
- Steepest learning curve of all frameworks
- Overkill for simple strategy research
- Documentation is improving but still lags the feature set

**Installation**:
```bash
pip install nautilus_trader
```

---

### Backtrader (Legacy)

**GitHub**: [mementum/backtrader](https://github.com/mementum/backtrader) | **Stars**: ~15k | **Last meaningful commit**: 2021

**Honest assessment**: Backtrader was dominant from 2015-2021 and has an enormous tutorial ecosystem. It still works. However, the author has not maintained it since 2021, and there are significant open issues and compatibility problems with recent Python versions (3.11+). The community fork [backtrader2](https://github.com/AB-CE/backtrader2) attempts to maintain compatibility.

**Recommendation**: If you have existing Backtrader code, it likely still works — don't rewrite it. For new projects in 2026, choose a different framework. The massive tutorial ecosystem (YouTube, blogs) is a genuine advantage for learners, but the codebase itself is not moving forward.

---

## Critical Pitfalls in Backtesting

Understanding these pitfalls is more important than choosing the right framework. A strategy that backtest's perfectly but fails live is almost always due to one of these:

### 1. Look-Ahead Bias (Most Common)

Using data in your signal that wasn't available at the time of the trade.

```python
# WRONG — uses closing price to generate the signal, but you can't trade AT the close
if today_close > yesterday_close * 1.02:
    buy_at_close()  # The price that determined the signal IS the execution price

# CORRECT — use yesterday's data to trade today's open
if yesterday_close > day_before_close * 1.02:
    buy_at_todays_open()
```

### 2. Survivorship Bias

Running your strategy on today's S&P 500 components means you're excluding companies that went bankrupt or were delisted. Your backtest is implicitly picking only survivors.

**Fix**: Use point-in-time index constituent data. Sources:
- Sharadar (Nasdaq Data Link) — paid, comprehensive
- `s&p500-historical-components` datasets on Kaggle — free, spot-check for accuracy

### 3. Overfitting (Optimization Bias)

If you optimize 20 parameters and test on the same data you optimized on, you're fitting to noise.

```python
# Correct workflow:
# 1. Split data: 70% in-sample, 30% out-of-sample (walk-forward)
# 2. Optimize ONLY on in-sample data
# 3. Evaluate ONCE on out-of-sample
# 4. If you peek at out-of-sample, it's contaminated

# Walk-forward validation example
train = data['2010':'2019']
test  = data['2020':'2024']
# Optimize on train, validate on test — NEVER go back and optimize again
```

### 4. Ignoring Transaction Costs

Even 0.1% round-trip commission destroys high-frequency strategies. Model realistic costs:

```python
# Realistic commission tiers (2026)
# Retail broker (e.g., Interactive Brokers IBKR Lite): ~$0 commission, but payment for order flow
# IBKR Pro: $0.005/share, minimum $1.00/order
# ETF trading: Effectively $0 at major brokers, but bid-ask spread matters

# Rule of thumb: if your strategy trades weekly or less, commission impact is small.
# If it trades daily or more, model commission explicitly.
commission_per_trade = 0.002  # 0.2% round-trip — conservative for retail
```

### 5. Ignoring Slippage & Market Impact

Your backtest assumes you trade at exactly the open or close price. In reality:
- You rarely get the exact open/close
- Large orders move the market against you

```python
# Simple slippage model
slippage_pct = 0.001  # 0.1% average slippage per trade

# In backtesting.py:
bt = Backtest(data, strategy, cash=10_000,
              commission=.002,
              exclusive_orders=True)  # Prevents order stacking
```

### 6. Unrealistic Fill Assumptions

Assuming 100% fill at the target price is unrealistic for illiquid stocks or large positions.

---

## Performance Metrics Libraries

These libraries compute portfolio statistics and work with any backtest output (just pass a returns Series):

**[QuantStats](https://github.com/ranaroussi/quantstats)**
```bash
pip install quantstats
```
```python
import quantstats as qs
returns = portfolio_value.pct_change().dropna()
qs.reports.html(returns, benchmark='SPY', output='report.html')
# Generates full tearsheet: Sharpe, Sortino, max drawdown, rolling stats, monthly returns heatmap
```

**[pyfolio](https://github.com/quantopian/pyfolio)** (Quantopian legacy, still functional)
```bash
pip install pyfolio-reloaded  # maintained fork
```

**[empyrical](https://github.com/quantopian/empyrical)**
```python
pip install empyrical
import empyrical
sharpe = empyrical.sharpe_ratio(returns)
max_dd = empyrical.max_drawdown(returns)
calmar = empyrical.calmar_ratio(returns)
```

**Manual metrics** (when you want to understand the math):
```python
import numpy as np
import pandas as pd

def compute_metrics(returns: pd.Series, risk_free_rate: float = 0.05) -> dict:
    """Compute key portfolio performance metrics."""
    ann_factor = 252  # Trading days per year

    excess = returns - risk_free_rate / ann_factor
    sharpe = (excess.mean() / returns.std()) * np.sqrt(ann_factor)

    rolling_max = (1 + returns).cumprod().cummax()
    drawdown = ((1 + returns).cumprod() / rolling_max) - 1
    max_drawdown = drawdown.min()

    total_return = (1 + returns).prod() - 1
    years = len(returns) / ann_factor
    cagr = (1 + total_return) ** (1 / years) - 1

    sortino = (excess.mean() / returns[returns < 0].std()) * np.sqrt(ann_factor)

    return {
        "CAGR": f"{cagr:.2%}",
        "Sharpe Ratio": f"{sharpe:.2f}",
        "Sortino Ratio": f"{sortino:.2f}",
        "Max Drawdown": f"{max_drawdown:.2%}",
        "Total Return": f"{total_return:.2%}",
    }
```

---

## Data Sources for Backtesting

| Use Case | Best Free Source | Best Paid Source |
|----------|-----------------|-----------------|
| US equity daily EOD | Tiingo, Stooq | Polygon.io, Refinitiv |
| Adjusted for splits/dividends | Tiingo (auto-adjusted) | Polygon.io |
| Point-in-time fundamentals | SEC EDGAR XBRL | Sharadar (Nasdaq Data Link) |
| S&P 500 historical constituents | Kaggle datasets (verify accuracy) | Compustat |
| Options historical chains | CBOE Datashop | Polygon.io, OptionsDX |
| Intraday (minute bars) | Limited free options | Polygon.io, Refinitiv |

See [awesome-financial-data-apis](../awesome-financial-data-apis/README.md) for full details on each source.

---

## Decision Guide: Which Framework to Use?

```
Are you just getting started with backtesting?
  └─ YES → backtesting.py

Do you need to test thousands of parameter combinations quickly?
  └─ YES → vectorbt (open source) or vectorbtpro (paid)

Do you need institutional-grade realism (options, futures, multi-asset)?
  └─ YES → QuantConnect LEAN

Do you need live trading capability?
  └─ YES → Are you willing to invest 2-4 weeks learning the system?
      ├─ YES (want best performance) → nautilus_trader
      └─ YES (want broader community) → QuantConnect LEAN

Are you building portfolio-level strategies (allocation, rebalancing)?
  └─ YES → bt

Do you have existing Backtrader code?
  └─ YES → Keep using it. Don't rewrite unless you hit hard limits.
  └─ NO → Do not start a new project with Backtrader in 2026.
```

---

## Learning Resources

### Books
- *Advances in Financial Machine Learning* — Marcos López de Prado (2018). Quantitative strategy development; addresses pitfalls rigorously.
- *Quantitative Trading* — Ernest Chan (2008, updated). Practical strategy development process.
- *Python for Finance* — Yves Hilpisch (O'Reilly). Python-first approach to financial modeling.

### Courses
- QuantConnect Learning Center — Free, hands-on LEAN tutorials
- Quantra (QuantInsti) — Paid but comprehensive Python algorithmic trading courses

### Communities
- [r/algotrading](https://www.reddit.com/r/algotrading/) — Active community; excellent wiki
- QuantConnect Community Forum — Framework-specific questions
- [QuantLib mailing list](https://lists.sourceforge.net/lists/listinfo/quantlib-users) — For derivatives pricing

### Research Papers (Free)
- Fama & French factor papers (available on Ken French's Dartmouth website)
- AQR Capital's whitepapers — Free, high-quality quantitative research
- SSRN Financial Economics Network — Pre-publication academic papers

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). When adding a framework or library:
- Verify it is still actively maintained (last commit within 18 months)
- Include your own test results, not just marketing claims
- Note any known compatibility issues with current Python versions

---

## License

[CC0 1.0 Universal](LICENSE) — Public domain.

*This repository is maintained by independent researchers. No framework listed here is a commercial partner or sponsor.*
