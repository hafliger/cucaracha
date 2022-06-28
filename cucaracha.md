// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © juanhafliger

//@version=5
indicator(title = "CUCARACHA - v4.1", shorttitle="CUCARACHAv4.1", overlay=true)

//==================
// RSI
//------------------
rsi = ta.rsi(close, 21)

//==================
// STOCHASTIC
//------------------
stochD = ta.stoch(close, high, low, 21)

//==================
// VWAP
//------------------
[vwap, upper, lower] = ta.vwap(open, 15, 1.0)

//==================
// EMAS 9 AND 90
//------------------
slowlenght = input.int(defval=90, minval=1, title="Média Lenta")
fastlenght = input.int(defval=9, minval=1, title="Média Rápida")

slowEMA = ta.ema(close, slowlenght)
fastEMA = ta.ema(close, fastlenght)

plot(slowEMA, title="SLOW EMA", color=color.red, linewidth=2)
plot(fastEMA, title="FAST EMA", color=color.green, linewidth=2)

//==================
// MACD 
//------------------
//Inputs
fastLen = input.int(defval=12, minval=1, title="Média Rápida")
slowLen = input.int(defval=26, minval=1, title="Média lenta")
signalLen = input.int(defval=9, minval=1, title="Sinal")

// MACD conditions
fast_ma = ta.ema(close, fastLen)
slow_ma = ta.ema(close, slowLen)
macd = fast_ma - slow_ma
signal = ta.ema(macd, signalLen)
macdHistogram = macd - signal

buyMACD = ta.crossover(macd, signal) 
sellMACD = ta.crossunder(macd, signal)
classOfMACD = macdHistogram > 0 ? "A" : "B"

// PLOT THE SIGNAL MACD
//------------------
buy = buyMACD and signal > 0
sell = sellMACD and signal < 0

plotshape(buy, title="BUY MACD CROSS", text="", location=location.abovebar, style=shape.cross, size=size.small, color=color.green, textcolor=color.white)
plotshape(sell, title="SELL MACD CROSS",text="", location=location.belowbar, style=shape.cross, size=size.small, color=color.red, textcolor=color.white)

//==================
// PLOT THE SIGNAL PRICE EXHAUSTION
//------------------
buyX = rsi < 30 and stochD < 20 and close < lower and macdHistogram < -0.001 and close < lower
sellX = rsi > 70 and stochD > 80 and close > upper and macdHistogram > 0.001 and close > upper

plotshape(buyX, title="BUY EXHAUSTION", text="", location=location.belowbar, style=shape.triangleup, size=size.tiny, color=color.green, textcolor=color.white)
plotshape(sellX, title="SELL EXHAUSTION",text="", location=location.abovebar, style=shape.triangledown, size=size.tiny, color=color.red, textcolor=color.white)
//===========

//==================
// LINEAR REGRESSION
//------------------

lengthRL = input.int(defval=50, minval=1, title="ATR") 
offset = 0 // Offset
src = close // Source
lsma = ta.linreg(src, lengthRL, offset) //
lsma2 = ta.linreg(lsma, lengthRL, offset)
eq= lsma-lsma2
deltaRL = lsma+eq

// plot(deltaRL, color=#fffffF4a, linewidth=3, style=plot.style_stepline)


length = 1 // Periodo ATR
mult = 2.3 // Periodo Multiplicador
showLabels = true // Exibe ou nao o HINT conforme o BUY e SELL Signal

atr = mult * ta.atr(length)
 
longStop = ta.highest(close, length) - atr
longStopPrev = nz(longStop[1], longStop) 
longStop := close[1] > longStopPrev ? math.max(longStop, longStopPrev) : longStop

shortStop = ta.lowest(close, length) + atr
shortStopPrev = nz(shortStop[1], shortStop)
shortStop := close[1] < shortStopPrev ? math.min(shortStop, shortStopPrev) : shortStop

var int dir = 1
dir := close > shortStopPrev ? 1 : close < longStopPrev ? -1 : dir

var color longColor = color.green
var color shortColor = color.red

setupForSell = ((close < fastEMA and close < slowEMA) and fastEMA < slowEMA) 
setupForBuy = ((close > fastEMA and close > slowEMA) and fastEMA > slowEMA) 

messageSell = "Sell signal for {{ticker}}"
messageBuy = "Buy signal for {{ticker}}"

buySignal = dir == 1 and dir[1] == -1 and setupForBuy
plotshape(buySignal ? longStop : na, title="BUY RECOMENDATION", text="BUY", location=location.belowbar, style=shape.labelup, size=size.tiny, color=longColor, textcolor=color.white)

sellSignal = dir == -1 and dir[1] == 1 and setupForSell
plotshape(sellSignal ? shortStop : na, title="SELL RECOMENDATION", text="SELL" , location=location.abovebar, style=shape.labeldown, size=size.tiny, color=shortColor, textcolor=color.white)

alertcondition(buySignal, title="BUY SIGNAL", message=messageBuy)
alertcondition(sellSignal, title="SELL SIGNAL", message=messageSell)
