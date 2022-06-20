// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © juanhafliger

//@version=5
indicator(title = "CUCARACHA - SETUP A/B + BUY e SELL signal", shorttitle="CUCARACHA", overlay=true)

// EMAS 
// Medias Móveis Exponenciais de 9 e 90 para o SETUP A / SETUP B e Pernadas
// em conjunto com MACD nos ajuda a visualizar se há um momentum vendedor ou comprador
slowlenght = input.int(defval=90, minval=1, title="Média Lenta")
fastlenght = input.int(defval=9, minval=1, title="Média Rápida")

slowEMA = ta.ema(close, slowlenght)
fastEMA = ta.ema(close, fastlenght)

plot(slowEMA, title="EMA Lenta", color=color.red, linewidth=2)
plot(fastEMA, title="EMAP Rápida", color=color.green, linewidth=2)



// CURVA DE REGRESSÃO LINEAR
// Curva de regressão linear. Uma linha que melhor se adapta aos preços especificados 
// ao longo de um período de tempo definido pelo usuário. Ela é calculada usando o método 
// dos mínimos quadrados. O resultado desta função é calculado utilizando a fórmula: 
// linreg = intercept + slope * (length - 1 - offset)
// onde intercepção e inclinação são os valores calculados com o método dos mínimos quadrados na série `source'.
// se ha uma relacao ascendente entre as duas variaves temos um vieis de alta
// se há uma relacao descendente entre as duvas variáveis temos um vies de baixa

lengthRL = input.int(defval=50, minval=1, title="ATR") // Comprimento
offset = 0 // Offset
src = close // Source
lsma = ta.linreg(src, lengthRL, offset) //
lsma2 = ta.linreg(lsma, lengthRL, offset)
eq= lsma-lsma2
deltaRL = lsma+eq

plot(deltaRL, color=#fffffF4a, linewidth=3, style=plot.style_stepline)

// ATR - Averange True Range
// média de amplitude de variação (muito usado em FOREX)
// retorna o RMA do alcance verdadeiro. 
// O alcance verdadeiro é máximo (máxima - mínima, abs(máxima - fechamento[1]), abs(mínima - fechamento [1])).
// Mede a volatidade, levando em conta qualquer gap no movimento do preço
// o calculo do atr é baseado em 14 periodos, mas aqui usaremos 1

length = 1 // Periodo ATR
mult = 2.0 // Periodo Multiplicador
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

buySignal = dir == 1 and dir[1] == -1
plotshape(buySignal ? longStop : na, title="Recomendação de compra", text="Compra", location=location.belowbar, style=shape.labelup, size=size.tiny, color=longColor, textcolor=color.white)

sellSignal = dir == -1 and dir[1] == 1
plotshape(sellSignal ? shortStop : na, title="Recomendação de venda", text="Venda", location=location.abovebar, style=shape.labeldown, size=size.tiny, color=shortColor, textcolor=color.white)

changeCond = dir != dir[1]
alertcondition(changeCond, title="Alerta: Mudança de Direção", message="A saída da vela mudou de direção!")
alertcondition(buySignal, title="Alerta: Viés de Compra", message="Recomendação para sair de Venda!")
alertcondition(sellSignal, title="Alerta: Viés de Venda", message="Recomendação para sair de Compra!")


