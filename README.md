# Multi-Timeframe Structure + Breakout Strategy

## Strategy Idea

This strategy combines higher timeframe trend direction with lower timeframe structure breakouts. Instead of trading every breakout, it aligns entries with the broader market direction and waits for confirmation from recent price structure.

The goal is to improve trade selection by filtering signals through both trend context and local market behavior.

## How It Works

### 1. Higher Timeframe Trend Filter

A higher timeframe EMA is used to define the overall direction.

Price above HTF EMA → bullish bias

Price below HTF EMA → bearish bias

This helps avoid trades against the dominant trend.

### 2. Market Structure Levels

The script tracks recent highs and lows over a lookback period.

Highest high → resistance reference

Lowest low → support reference

These levels act as structure zones.

### 3. Breakout Confirmation

Trades are triggered only when price breaks these structure levels:

Break above recent high → potential continuation upward

Break below recent low → potential continuation downward

### 4. Volatility-Based Risk Management

Stop-loss and take-profit levels are calculated using ATR.

Stop-loss adapts to volatility

Risk-to-reward ratio defines the target

## Key Features

Multi-timeframe trend alignment

Structure-based breakout entries

ATR-based adaptive exits

Non-repainting logic

Designed for backtesting and experimentation

## When It May Perform Better

Trending markets

Volatility expansion phases

Higher timeframe directional moves

It may produce fewer signals in low-volatility or sideways conditions.

## Important Note

This strategy is intended for testing and learning purposes. Results vary depending on market conditions, timeframe, and parameter selection. Always validate across multiple instruments before using it in live environments.

## Automate It with PineGen AI

Take this strategy to the next level with PineGen AI automation:

Website: https://www.pinegen.ai/

Twitter: https://x.com/PineGenAI

Telegram Channel: https://t.me/PineGenAI

YouTube Channel: https://www.youtube.com/@pinegenai

## Pine Script v6 Strategy Code

```pine
//@version=6
strategy(
    "MTF Structure Breakout Strategy",
    overlay = true,
    initial_capital = 100000,
    default_qty_type = strategy.percent_of_equity,
    default_qty_value = 3,
    commission_type = strategy.commission.percent,
    commission_value = 0.05,
    slippage = 1
)

// ───── INPUTS ─────
htfTF     = input.timeframe("60", "Higher Timeframe")
htfEmaLen = input.int(100, "HTF EMA Length")

lookback  = input.int(20, "Structure Lookback")
atrLen    = input.int(14, "ATR Length")
atrMult   = input.float(1.8, "ATR Stop Multiplier")
rr        = input.float(2.0, "Risk Reward")

// ───── HIGHER TIMEFRAME TREND ─────
htfEMA = request.security(
     syminfo.tickerid,
     htfTF,
     ta.ema(close, htfEmaLen),
     lookahead = barmerge.lookahead_off
)

trendUp   = close > htfEMA
trendDown = close < htfEMA

// ───── STRUCTURE LEVELS ─────
recentHigh = ta.highest(high, lookback)
recentLow  = ta.lowest(low, lookback)

// ───── BREAKOUT CONDITIONS (GLOBAL SAFE) ─────
breakUp   = ta.crossover(close, recentHigh[1])
breakDown = ta.crossunder(close, recentLow[1])

// ───── ENTRY CONDITIONS ─────
longCondition  = breakUp and trendUp
shortCondition = breakDown and trendDown

if longCondition and strategy.position_size == 0
    strategy.entry("Long", strategy.long)

if shortCondition and strategy.position_size == 0
    strategy.entry("Short", strategy.short)

// ───── RISK MANAGEMENT ─────
atrVal = ta.atr(atrLen)

longStop   = strategy.position_avg_price - atrVal * atrMult
longTarget = strategy.position_avg_price + atrVal * atrMult * rr

shortStop   = strategy.position_avg_price + atrVal * atrMult
shortTarget = strategy.position_avg_price - atrVal * atrMult * rr

strategy.exit("Exit Long", from_entry="Long", stop=longStop, limit=longTarget)
strategy.exit("Exit Short", from_entry="Short", stop=shortStop, limit=shortTarget)

// ───── VISUALS ─────
plot(htfEMA, title="HTF EMA", color=color.orange)
plot(recentHigh, title="Structure High", color=color.green)
plot(recentLow, title="Structure Low", color=color.red)
```
