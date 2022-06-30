
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © juanhafliger

//@version=5
indicator(title = "SamucaTrader Singal 0.0.1", shorttitle="SamucaSignal", overlay=true)

// init values
base = -1

// sections
var SectionInfos = "SINAL DO ATIVO"
var SectionEntryValues = "Valor de Entrada"
var SectionTargetsValues = "Alvos"
var SectionAlarm = "ATIVAÇÃO DO DISPARO"

longShortOption = input.string(defval="LONG", title="Mod", options=["LONG","SHORT", "SPOT"] , inline = "11", group = SectionInfos)
exchangeOption = syminfo.prefix
coinOption = syminfo.ticker
currentPrice = str.tostring(close)
investOption = "10%" //input.string(defval="5%", title="Invt", options=["5%","10%", "15%", "20%"] , inline = "22", group = SectionInfos)
levarageOption = input.string(defval="5x", title="X", options=["3x", "5x","10x", "20x", "30x", "40x", "Sem alavancagem"] , inline = "11", group = SectionInfos)
entryMinOption = input.float(defval=base, title="Min", inline = "33", group = SectionEntryValues)
entryMaxOption = input.float(defval=base, title="Max", inline = "33", group = SectionEntryValues)
stopLossOption = input.float(defval=base, title="STOP LOSS", group = SectionTargetsValues)
target1Option = input.float(defval=base, title="Alvo 1", group = SectionTargetsValues)
target2Option = input.float(defval=base, title="Alvo 2", group = SectionTargetsValues)
target3Option = input.float(defval=base, title="Alvo 3", group = SectionTargetsValues)
target4Option = input.float(defval=base, title="Alvo 4", group = SectionTargetsValues)
target5Option = input.float(defval=base, title="Alvo 5", group = SectionTargetsValues)
target6Option = input.float(defval=base, title="Alvo 6", group = SectionTargetsValues)

trigerValue = input.float(defval=-1, title="No preço",  inline="55", group = SectionAlarm)
activateSignal = input.bool(defval=false, title="Imediato!", inline="55", group = SectionAlarm)
chartOption = input(defval="", title="Análise", inline="11", group=SectionInfos)

iconStategy = longShortOption == "SHORT" ? "🐻":"🐮" 
longShortText = "\n  "+iconStategy+" *Estratégia*: *"+longShortOption+"*"
exchangeText = "\n  👉 *Exchange*: *"+str.tostring(exchangeOption)+"*"
coinText = "\n  👉 *Coin*: *"+coinOption+"*"
investText = "\n  👉 *Investir*: *"+investOption+"*"
levarageText = "\n  👉 *Alavancagem*: *"+levarageOption+"*" 
entryText = "\n  💰 *Entrada*: *"+str.tostring(entryMinOption)+"* / *"+str.tostring(entryMaxOption)+"*"
stopLossText = stopLossOption != -1 ? "\n  🚫 *Stop*: --- *"+str.tostring(stopLossOption)+"*" : "\n  🚫 *Stop*: --- Em alguns instantes"
textTarget1 = target1Option != -1 ? "\n ⎿  Alvo 1 --- " +  str.tostring(target1Option): na
textTarget2 = target2Option != -1 ? "\n ⎿  Alvo 2 --- " +  str.tostring(target2Option): na
textTarget3 = target3Option != -1 ? "\n ⎿  Alvo 3 --- " +  str.tostring(target3Option): na
textTarget4 = target4Option != -1 ? "\n ⎿  Alvo 4 --- " +  str.tostring(target4Option): na
textTarget5 = target5Option != -1 ? "\n ⎿  Alvo 5 --- " +  str.tostring(target5Option): na
textTarget6 = target6Option != -1 ? "\n ⎿  Alvo 6 --- " +  str.tostring(target6Option): na
textChart = chartOption != "" ? "\n  📊 Análise: "+chartOption : na
alertMessage = "\n --- ⌁ --- \n \n 🤖 Fala pessoal! Temos um novo sinal! 💰💰💰\n \n --- ⌁ --- \n "+longShortText+""+exchangeText+""+coinText+""+investText+""+levarageText+""+entryText+""+stopLossText+"\n  🎯 Alvos "+textTarget1+""+textTarget2+""+textTarget3+""+textTarget4+""+textTarget5 +""+textTarget6+"\n  ⚠️ Preço atual: "+currentPrice+" \n \n --- ⌁ --- \n 💡Avalie o timeframe acima e abaixo antes de fazer uma entrada. \n 💡Nunca faça uma entrada sem definir seu STOP LOSS.\n --- ⌁ --- \n  "+ textChart 

sendsignal = activateSignal or (ta.cross(close, trigerValue))

if sendsignal
    alert(message=alertMessage, freq=alert.freq_once_per_bar_close)
