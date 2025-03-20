# Golden Strategy - (Pine Script)

## Overview
The **Golden Strategy** is a backtestable trading strategy written in Pine Script (version 6) for TradingView. This strategy combines multiple technical indicators to identify potential entry and exit points for trades. The strategy utilizes:

- **Exponential Moving Average (EMA) & Simple Moving Average (SMA)**: Determines trend direction using crossover signals.
- **MACD (Moving Average Convergence Divergence)**: Confirms bullish or bearish momentum.
- **RSI (Relative Strength Index)**: Helps identify overbought and oversold conditions.
- **Volume Condition**: Ensures trades are executed during significant market activity.
- **Fixed Stop Loss & Take Profit**: Implements risk management through pre-defined stop loss and take profit percentages.
- **Risk-Based Position Sizing**: Calculates trade size based on risk percentage of total equity.

## Strategy Components

### 1. Backtest Time Frame
Defines the start and end period for backtesting:

- **From Month/Year**: The starting point of the backtest.
- **To Month/Year**: The ending point, set dynamically to three months ahead.

```pinescript
Start = timestamp(FromY, FromM, 00, 00)
End = timestamp(ToY, ToM, 23, 59)
```

### 2. User Inputs
Various parameters configurable by the user:

- **EMA Length**: Short-term moving average.
- **SMA Length**: Long-term moving average.
- **MACD Settings**: Fast, slow, and signal line lengths.
- **RSI Settings**: RSI period and thresholds for overbought/oversold conditions.
- **Stop Loss & Take Profit**: Fixed percentages to manage risk.
- **Risk per Trade**: Percentage of equity risked on each trade.
- **Volume SMA Length**: Determines volume conditions.

### 3. Indicator Calculations
Computes values for EMA, SMA, MACD, RSI, and Volume SMA:

```pinescript
emaValue = ta.ema(close, emaLength)
smaValue = ta.sma(close, smaLength)
[macdLine, macdSignal, macdHist] = ta.macd(close, fastLengthMACD, slowLengthMACD, sigLengthMACD)
rsiValue = ta.rsi(close, rsiLength)
volumeAvg = ta.sma(volume, volumeLength)
```

### 4. Entry Conditions
Defines logic for entering long and short positions:

- **Long Entry**: When EMA crosses above SMA, MACD is bullish, and volume is high.
- **Short Entry**: When EMA crosses below SMA, MACD is bearish, and volume is high.

```pinescript
enterLongCond = bullishCross and upTrendMACD and volumeCond
enterShortCond = bearishCross and downTrendMACD and volumeCond
```

### 5. Exit Conditions
Defines logic for closing trades:

- **Long Exit**: When RSI falls below the lower band or MACD turns bearish.
- **Short Exit**: When RSI exceeds the upper band or MACD turns bullish.

```pinescript
exitLongCond = (rsiValue < rsiLowerBand) or (macdLine < macdSignal)
exitShortCond = (rsiValue > rsiUpperBand) or (macdLine > macdSignal)
```

### 6. Position Sizing
Automatically calculates position size based on risk percentage:

```pinescript
f_calcPositionSize(stopLossPerc) =>
    riskCapital = strategy.equity * (riskPerTrade / 100.0)
    stopDist = close * (stopLossPerc / 100.0)
    positionSize = riskPerTrade > 0 and stopLossPerc > 0 ? riskCapital / stopDist : 1.0
    positionSize
```

### 7. Strategy Execution
Implements trade execution and risk management:

- Enters trades based on entry conditions.
- Applies stop loss and take profit levels if enabled.
- Closes trades based on exit conditions.

```pinescript
if (time >= Start and time <= End) 
    if enterLongCond 
        strategy.entry(id="Long", direction=strategy.long, qty=f_calcPositionSize(fixStopLossPerc))
    
    if enterShortCond 
        strategy.entry(id="Short", direction=strategy.short, qty=f_calcPositionSize(fixStopLossPerc))
    
    if strategy.position_size > 0 and useFixedSLTP
        strategy.exit(id="SL/TP Long", from_entry="Long", stop=strategy.position_avg_price * (1.0 - fixStopLossPerc / 100.0), limit=strategy.position_avg_price * (1.0 + fixTakeProfitPerc / 100.0))
    
    if strategy.position_size < 0 and useFixedSLTP
        strategy.exit(id="SL/TP Short", from_entry="Short", stop=strategy.position_avg_price * (1.0 + fixStopLossPerc / 100.0), limit=strategy.position_avg_price * (1.0 - fixTakeProfitPerc / 100.0))
    
    if exitLongCond and strategy.position_size > 0 
        strategy.close("Long", comment="Exit Long Condition Met")
    
    if exitShortCond and strategy.position_size < 0 
        strategy.close("Short", comment="Exit Short Condition Met")
```

## Summary
The **Golden Strategy** aims to provide a well-balanced, rule-based trading approach by integrating multiple technical indicators. It ensures:

- Clear trend identification with EMA/SMA crossovers.
- Confirmation using MACD momentum.
- Volume-based filtering for reliable entries.
- RSI-based exits to optimize trade timing.
- Fixed risk management through SL/TP and position sizing.

This strategy is suitable for traders who seek a structured and systematic approach to trading financial markets. Backtesting is recommended before live trading to fine-tune the parameters for different market conditions.
