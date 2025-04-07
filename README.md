//@version=5
indicator("Elite Reversal Signal", overlay=true, precision=2)

lookback = input.int(5, "Swing Lookback", minval=1)
wickThreshold = input.float(1.5, "Wick Multiplier (Wick/Candle Body)", step=0.1)
useEngulfing = input.bool(true, "Use Engulfing Confirmation")
showArrows = input.bool(true, "Show Reversal Arrows")

// Swing High/Low Detection (Fixed)
isSwingHigh = high[lookback] > ta.highest(high, lookback * 2 + 1)[lookback] and high[lookback] == high
isSwingLow  = low[lookback] < ta.lowest(low, lookback * 2 + 1)[lookback] and low[lookback] == low

// Current Candle Properties
bodySize    = math.abs(close - open)
upperWick   = high - math.max(close, open)
lowerWick   = math.min(close, open) - low

// Liquidity Sweep Detection
liquiditySell = isSwingHigh and upperWick[lookback] > bodySize[lookback] * wickThreshold
liquidityBuy  = isSwingLow  and lowerWick[lookback] > bodySize[lookback] * wickThreshold

// Engulfing Confirmation
bullishEngulfing = close > open and close > close[1] and open < close[1]
bearishEngulfing = close < open and close < close[1] and open > close[1]

buyConfirm  = liquidityBuy[1]  and (not useEngulfing or bullishEngulfing)
sellConfirm = liquiditySell[1] and (not useEngulfing or bearishEngulfing)

// Plot Arrows
plotshape(showArrows and buyConfirm, title="Buy Signal", style=shape.arrowup, location=location.belowbar, color=color.black, size=size.small, offset=-1)
plotshape(showArrows and sellConfirm, title="Sell Signal", style=shape.arrowdown, location=location.abovebar, color=color.fuchsia, size=size.small, offset=-1)

// Alerts
alertcondition(buyConfirm, title="Buy Alert", message="Elite Reversal Buy Signal!")
alertcondition(sellConfirm, title="Sell Alert", message="Elite Reversal Sell Signal!")
