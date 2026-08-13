# Npro
Pro
//@version=6
indicator("XAUUSD PRO Smart Signals - 3 Engines", shorttitle="XAU PRO 3E", overlay=true, max_labels_count=500, max_lines_count=500, max_boxes_count=500)

//=====================================================================
// XAUUSD PRO SMART SIGNALS - FINAL
//
// ENGINE 1 : REVERSAL PATTERNS
// ENGINE 2 : SPECIAL PATTERNS
// ENGINE 3 : SUPPORT & RESISTANCE
//
// 2 engines in the same direction = signal + GOLD circle
// 3 engines in the same direction = signal + GOLD circle + PINK circle
//
// IMPORTANT:
// The engine rules below are kept unchanged.
// Only the visual interface has been adjusted:
// - Dark blue chart background.
// - Green BUY signal with integrated upward triangle.
// - Red SELL signal with integrated downward triangle.
// - Arabic signal text in white and larger.
// - Gold/Pink confirmation circles remain outside the signal.
// - No additional signal is generated merely because a circle exists.
// - Same-direction repeated signals remain blocked.
//=====================================================================


//=====================================================================
// 1. GENERAL SETTINGS
//=====================================================================
groupGeneral = "واجهة المؤشر"

showSignals = input.bool(true, "إظهار إشارات الشراء والبيع", group=groupGeneral)
showConfirmations = input.bool(true, "إظهار دوائر التأكيد", group=groupGeneral)
showStatistics = input.bool(true, "إظهار مربع الإحصائيات", group=groupGeneral)
showMovingAverage = input.bool(true, "إظهار خط الموفينج", group=groupGeneral)

emaLength = input.int(200, "طول الموفينج", minval=20, maxval=500, group=groupGeneral)


//=====================================================================
// 2. BACKGROUND
//=====================================================================
groupBackground = "خلفية الرسم البياني"

// Dark blue only.
// It is intentionally not navy and not cyan.
backgroundColor = input.color(
     color.rgb(18, 45, 105),
     "لون خلفية الرسم البياني",
     group=groupBackground)

bgcolor(backgroundColor, title="خلفية الرسم البياني")


//=====================================================================
// 3. CANDLE COLORS
//=====================================================================
bullCandleColor = color.rgb(0, 255, 70)
bearCandleColor = color.rgb(255, 35, 35)

barcolor(
     close > open ? bullCandleColor :
     close < open ? bearCandleColor :
     na,
     title="ألوان الشموع")


//=====================================================================
// 4. COMMON DATA
//=====================================================================
atr = ta.atr(14)

candleRange = math.max(high - low, syminfo.mintick)
body = math.abs(close - open)

upperWick = high - math.max(open, close)
lowerWick = math.min(open, close) - low

bull = close > open
bear = close < open

doji = body <= candleRange * 0.10

confirmed = barstate.isconfirmed

ema200 = ta.ema(close, emaLength)

emaBull = close > ema200
emaBear = close < ema200

movingColor = emaBull ? color.yellow : color.rgb(255, 100, 100)

plot(
     showMovingAverage ? ema200 : na,
     title="Moving Average",
     color=movingColor,
     linewidth=2)


//=====================================================================
// 5. TIMEFRAME CONTROL
//=====================================================================
timeframeAllowed =
     not timeframe.isdaily and
     not timeframe.isweekly

reversalSpecialAllowed =
     timeframe.isminutes or
     timeframe.isseconds or
     (timeframe.isminutes and timeframe.multiplier <= 240)

engineAllowed = timeframeAllowed and reversalSpecialAllowed


//=====================================================================
// 6. PREVIOUS TREND
//=====================================================================
previousThreeBear =
     bear[1] and
     bear[2] and
     bear[3]

previousThreeBull =
     bull[1] and
     bull[2] and
     bull[3]

previousFourBear =
     previousThreeBear and bear[4]

previousFourBull =
     previousThreeBull and bull[4]

previousFiveBear =
     previousFourBear and bear[5]

previousFiveBull =
     previousFourBull and bull[5]

previousSixBear =
     previousFiveBear and bear[6]

previousSixBull =
     previousFiveBull and bull[6]

priorDownTrend =
     previousThreeBear or
     previousFourBear or
     previousFiveBear or
     previousSixBear

priorUpTrend =
     previousThreeBull or
     previousFourBull or
     previousFiveBull or
     previousSixBull


//=====================================================================
// 7. ENGINE 1 - REVERSAL PATTERNS
//=====================================================================

//---------------------------------------------------------------------
// Single candle reversal patterns
//---------------------------------------------------------------------
hammer =
     priorDownTrend and
     lowerWick >= body * 2.0 and
     upperWick <= math.max(body * 0.40, syminfo.mintick)

invertedHammer =
     priorDownTrend and
     upperWick >= body * 2.0 and
     lowerWick <= math.max(body * 0.40, syminfo.mintick)

dragonflyDoji =
     priorDownTrend and
     doji and
     lowerWick > upperWick * 2.0

bullMarubozu =
     priorDownTrend and
     bull and
     upperWick <= syminfo.mintick and
     lowerWick <= syminfo.mintick

shootingStar =
     priorUpTrend and
     body > 0 and
     upperWick >= body * 2.0 and
     lowerWick <= body * 0.40

hangingMan =
     priorUpTrend and
     body > 0 and
     lowerWick >= body * 2.0 and
     upperWick <= body * 0.40

gravestoneDoji =
     priorUpTrend and
     doji and
     upperWick > lowerWick * 2.0

bearMarubozu =
     priorUpTrend and
     bear and
     upperWick <= syminfo.mintick and
     lowerWick <= syminfo.mintick


//---------------------------------------------------------------------
// Two candle reversal patterns
//---------------------------------------------------------------------
bullEngulfing =
     priorDownTrend and
     bear[1] and
     bull and
     open <= close[1] and
     close >= open[1]

bearEngulfing =
     priorUpTrend and
     bull[1] and
     bear and
     open >= close[1] and
     close <= open[1]

bullHarami =
     priorDownTrend and
     bear[1] and
     bull and
     math.max(open, close) <= math.max(open[1], close[1]) and
     math.min(open, close) >= math.min(open[1], close[1])

bearHarami =
     priorUpTrend and
     bull[1] and
     bear and
     math.max(open, close) <= math.max(open[1], close[1]) and
     math.min(open, close) >= math.min(open[1], close[1])

piercingPattern =
     priorDownTrend and
     bear[1] and
     bull and
     close > (open[1] + close[1]) * 0.5 and
     close < open[1]

darkCloudCover =
     priorUpTrend and
     bull[1] and
     bear and
     close < (open[1] + close[1]) * 0.5 and
     close > open[1]

tweezerBottom =
     priorDownTrend and
     bear[1] and
     bull and
     math.abs(low - low[1]) <= atr * 0.15

tweezerTop =
     priorUpTrend and
     bull[1] and
     bear and
     math.abs(high - high[1]) <= atr * 0.15

bullCounterAttack =
     priorDownTrend and
     bear[1] and
     bull and
     math.abs(close - close[1]) <= atr * 0.10

bearCounterAttack =
     priorUpTrend and
     bull[1] and
     bear and
     math.abs(close - close[1]) <= atr * 0.10


//---------------------------------------------------------------------
// Three candle reversal patterns
//---------------------------------------------------------------------
morningStar =
     priorDownTrend and
     bear[2] and
     doji[1] and
     bull and
     close > (open[2] + close[2]) * 0.5

eveningStar =
     priorUpTrend and
     bull[2] and
     doji[1] and
     bear and
     close < (open[2] + close[2]) * 0.5

threeWhiteSoldiers =
     priorDownTrend and
     bull and
     bull[1] and
     bull[2] and
     close > close[1] and
     close[1] > close[2]

threeBlackCrows =
     priorUpTrend and
     bear and
     bear[1] and
     bear[2] and
     close < close[1] and
     close[1] < close[2]


//---------------------------------------------------------------------
// Special reversal patterns
//---------------------------------------------------------------------
bullKicker =
     priorDownTrend and
     bear[1] and
     bull and
     open > high[1]

bearKicker =
     priorUpTrend and
     bull[1] and
     bear and
     open < low[1]

bullBeltHold =
     priorDownTrend and
     bull and
     math.abs(open - low) <= syminfo.mintick

bearBeltHold =
     priorUpTrend and
     bear and
     math.abs(open - high) <= syminfo.mintick

matchingLow =
     priorDownTrend and
     bear[1] and
     bear and
     math.abs(close - close[1]) <= atr * 0.10

matchingHigh =
     priorUpTrend and
     bull[1] and
     bull and
     math.abs(close - close[1]) <= atr * 0.10

abandonedBabyBull =
     priorDownTrend and
     bear[2] and
     doji[1] and
     bull and
     low[1] < low[2] and
     low[1] < low

abandonedBabyBear =
     priorUpTrend and
     bull[2] and
     doji[1] and
     bear and
     high[1] > high[2] and
     high[1] > high

bullPinBar =
     priorDownTrend and
     body > 0 and
     lowerWick >= body * 2.5 and
     upperWick <= body * 0.20

bearPinBar =
     priorUpTrend and
     body > 0 and
     upperWick >= body * 2.5 and
     lowerWick <= body * 0.20

longLowerShadow =
     priorDownTrend and
     lowerWick >= math.max(body * 3.0, syminfo.mintick * 3)

longUpperShadow =
     priorUpTrend and
     upperWick >= math.max(body * 3.0, syminfo.mintick * 3)


//=====================================================================
// 8. ENGINE 1 FINAL DATABASE
//=====================================================================
bool reversalBuy = false
bool reversalSell = false
string reversalPattern = ""

if engineAllowed
    if hammer
        reversalBuy := true
        reversalPattern := "نموذج المطرقة"
    else if invertedHammer
        reversalBuy := true
        reversalPattern := "نموذج المطرقة المقلوبة"
    else if dragonflyDoji
        reversalBuy := true
        reversalPattern := "دوجي اليعسوب"
    else if bullMarubozu
        reversalBuy := true
        reversalPattern := "ماروبوزو صاعد"
    else if bullEngulfing
        reversalBuy := true
        reversalPattern := "الابتلاع الصاعد"
    else if bullHarami
        reversalBuy := true
        reversalPattern := "الحرامي الصاعد"
    else if piercingPattern
        reversalBuy := true
        reversalPattern := "نموذج الاختراق الصاعد"
    else if tweezerBottom
        reversalBuy := true
        reversalPattern := "القاع الملقاط"
    else if bullCounterAttack
        reversalBuy := true
        reversalPattern := "هجوم مضاد صاعد"
    else if morningStar
        reversalBuy := true
        reversalPattern := "نجمة الصباح"
    else if threeWhiteSoldiers
        reversalBuy := true
        reversalPattern := "الجنود الثلاثة"
    else if bullKicker
        reversalBuy := true
        reversalPattern := "كيكر صاعد"
    else if bullBeltHold
        reversalBuy := true
        reversalPattern := "حزام صاعد"
    else if matchingLow
        reversalBuy := true
        reversalPattern := "القاع المتطابق"
    else if abandonedBabyBull
        reversalBuy := true
        reversalPattern := "الطفل المتروك الصاعد"
    else if bullPinBar
        reversalBuy := true
        reversalPattern := "شمعة الدبوس الصاعدة"
    else if longLowerShadow
        reversalBuy := true
        reversalPattern := "الظل السفلي الطويل"
    else if shootingStar
        reversalSell := true
        reversalPattern := "الشهاب"
    else if hangingMan
        reversalSell := true
        reversalPattern := "الرجل المشنوق"
    else if gravestoneDoji
        reversalSell := true
        reversalPattern := "دوجي شاهد القبر"
    else if bearMarubozu
        reversalSell := true
        reversalPattern := "ماروبوزو هابط"
    else if bearEngulfing
        reversalSell := true
        reversalPattern := "الابتلاع الهابط"
    else if bearHarami
        reversalSell := true
        reversalPattern := "الحرامي الهابط"
    else if darkCloudCover
        reversalSell := true
        reversalPattern := "غطاء السحابة الداكنة"
    else if tweezerTop
        reversalSell := true
        reversalPattern := "القمة الملقاط"
    else if bearCounterAttack
        reversalSell := true
        reversalPattern := "هجوم مضاد هابط"
    else if eveningStar
        reversalSell := true
        reversalPattern := "نجمة المساء"
    else if threeBlackCrows
        reversalSell := true
        reversalPattern := "الغربان السوداء الثلاثة"
    else if bearKicker
        reversalSell := true
        reversalPattern := "كيكر هابط"
    else if bearBeltHold
        reversalSell := true
        reversalPattern := "حزام هابط"
    else if matchingHigh
        reversalSell := true
        reversalPattern := "القمة المتطابقة"
    else if abandonedBabyBear
        reversalSell := true
        reversalPattern := "الطفل المتروك الهابط"
    else if bearPinBar
        reversalSell := true
        reversalPattern := "شمعة الدبوس الهابطة"
    else if longUpperShadow
        reversalSell := true
        reversalPattern := "الظل العلوي الطويل"


//=====================================================================
// 9. ENGINE 2 - SPECIAL PATTERNS
//=====================================================================
specialMinBody = 0.20
shadowRatio = 2.0

specialShootingStar =
     priorUpTrend and
     body > 0 and
     body / candleRange <= 0.45 and
     upperWick >= body * shadowRatio and
     lowerWick <= body * 0.75 and
     close <= (high + low) * 0.5

specialBullLongShadow =
     priorDownTrend and
     lowerWick >= body * shadowRatio and
     lowerWick > upperWick and
     close >= (high + low) * 0.5

specialBearLongShadow =
     priorUpTrend and
     upperWick >= body * shadowRatio and
     upperWick > lowerWick and
     close <= (high + low) * 0.5

specialTweezerTop =
     priorUpTrend and
     bull[1] and
     bear and
     math.abs(high - high[1]) <= atr * 0.15

specialTweezerBottom =
     priorDownTrend and
     bear[1] and
     bull and
     math.abs(low - low[1]) <= atr * 0.15

judas =
     priorDownTrend and
     bear[1] and
     bull and
     body[1] >= candleRange[1] * specialMinBody and
     lowerWick >= body * 1.25 and
     close >= (high + low) * 0.5

inside212 =
     high[1] <= high[2] and
     low[1] >= low[2]

twoOneTwoBearish =
     priorUpTrend and
     bull[2] and
     inside212 and
     bear and
     close < low[1]

threeTwoTwoBullish =
     priorDownTrend and
     bear[2] and
     inside212 and
     bull and
     close > high[1]

insideBar =
     high <= high[1] and
     low >= low[1]

insideBarBullish =
     priorDownTrend and
     insideBar and
     bull

insideBarBearish =
     priorUpTrend and
     insideBar and
     bear

outsideBar =
     high >= high[1] and
     low <= low[1]

outsideBarBullish =
     priorDownTrend and
     outsideBar and
     bull and
     close > close[1]

outsideBarBearish =
     priorUpTrend and
     outsideBar and
     bear and
     close < close[1]

risingWindow =
     priorDownTrend and
     low > high[1] and
     low - high[1] >= atr * 0.05

fallingWindow =
     priorUpTrend and
     high < low[1] and
     low[1] - high >= atr * 0.05

specialThreeBlackCrows =
     priorUpTrend and
     bear and
     bear[1] and
     bear[2] and
     body >= candleRange * specialMinBody and
     body[1] >= candleRange[1] * specialMinBody and
     body[2] >= candleRange[2] * specialMinBody and
     close < close[1] and
     close[1] < close[2]

specialThreeSoldiers =
     priorDownTrend and
     bull and
     bull[1] and
     bull[2] and
     body >= candleRange * specialMinBody and
     body[1] >= candleRange[1] * specialMinBody and
     body[2] >= candleRange[2] * specialMinBody and
     close > close[1] and
     close[1] > close[2]

bullishGap =
     priorDownTrend and
     open > close[1] and
     open - close[1] >= atr * 0.05

bearishGap =
     priorUpTrend and
     open < close[1] and
     close[1] - open >= atr * 0.05


//=====================================================================
// 10. ENGINE 2 FINAL DATABASE
//=====================================================================
bool specialBuy = false
bool specialSell = false
string specialPattern = ""

if engineAllowed
    if threeTwoTwoBullish
        specialBuy := true
        specialPattern := "نمط 3-2-2 الصاعد"
    else if twoOneTwoBearish
        specialSell := true
        specialPattern := "نمط 2-1-2 الهابط"
    else if specialThreeBlackCrows
        specialSell := true
        specialPattern := "الغربان السوداء الثلاثة"
    else if specialThreeSoldiers
        specialBuy := true
        specialPattern := "الجنود الثلاثة"
    else if specialTweezerTop
        specialSell := true
        specialPattern := "الملقاط العلوي"
    else if specialTweezerBottom
        specialBuy := true
        specialPattern := "الملقاط السفلي"
    else if specialShootingStar
        specialSell := true
        specialPattern := "الشهاب"
    else if judas
        specialBuy := true
        specialPattern := "شمعة جوداس"
    else if risingWindow
        specialBuy := true
        specialPattern := "النافذة الصاعدة"
    else if fallingWindow
        specialSell := true
        specialPattern := "النافذة الهابطة"
    else if outsideBarBullish
        specialBuy := true
        specialPattern := "الشريط الخارجي الصاعد"
    else if outsideBarBearish
        specialSell := true
        specialPattern := "الشريط الخارجي الهابط"
    else if insideBarBullish
        specialBuy := true
        specialPattern := "الشريط الداخلي الصاعد"
    else if insideBarBearish
        specialSell := true
        specialPattern := "الشريط الداخلي الهابط"
    else if specialBullLongShadow
        specialBuy := true
        specialPattern := "الظل الطويل الصاعد"
    else if specialBearLongShadow
        specialSell := true
        specialPattern := "الظل الطويل الهابط"
    else if bullishGap
        specialBuy := true
        specialPattern := "الفجوة الصاعدة"
    else if bearishGap
        specialSell := true
        specialPattern := "الفجوة الهابطة"


//=====================================================================
// 11. ENGINE 3 - SUPPORT & RESISTANCE
//
// Each timeframe calculates its own pivots.
// No higher-timeframe S/R lines are imported.
//=====================================================================
groupSR = "محرك الدعم والمقاومة"

pivotLeft = input.int(3, "Pivot Left Bars", minval=1, maxval=20, group=groupSR)
pivotRight = input.int(3, "Pivot Right Bars", minval=1, maxval=20, group=groupSR)

zoneATR = input.float(0.20, "Same Zone ATR Multiplier", minval=0.01, step=0.01, group=groupSR)
rejectionATR = input.float(0.10, "Minimum Rejection ATR", minval=0.01, step=0.01, group=groupSR)

showSRLines = input.bool(true, "إظهار خطوط الدعم والمقاومة", group=groupSR)
maxSRLines = input.int(20, "Maximum Levels", minval=1, maxval=100, group=groupSR)

supportColor = color.rgb(0, 140, 255)
resistanceColor = color.rgb(255, 145, 0)

zoneTolerance = atr * zoneATR
minimumRejection = atr * rejectionATR

pivotHigh = ta.pivothigh(high, pivotLeft, pivotRight)
pivotLow = ta.pivotlow(low, pivotLeft, pivotRight)

var float previousHighPrice = na
var int previousHighBar = na

var float previousLowPrice = na
var int previousLowBar = na

var resistanceLines = array.new_line()
var supportLines = array.new_line()

var resistancePrices = array.new_float()
var supportPrices = array.new_float()

f_resistanceExists(float level) =>
    bool exists = false
    if array.size(resistancePrices) > 0
        for i = 0 to array.size(resistancePrices) - 1
            if math.abs(array.get(resistancePrices, i) - level) <= zoneTolerance
                exists := true
    exists

f_supportExists(float level) =>
    bool exists = false
    if array.size(supportPrices) > 0
        for i = 0 to array.size(supportPrices) - 1
            if math.abs(array.get(supportPrices, i) - level) <= zoneTolerance
                exists := true
    exists


//---------------------------------------------------------------------
// Resistance creation
//---------------------------------------------------------------------
if not na(pivotHigh)
    currentHighPrice = pivotHigh
    currentHighBar = bar_index - pivotRight

    if na(previousHighPrice)
        previousHighPrice := currentHighPrice
        previousHighBar := currentHighBar
    else
        sameZone = math.abs(currentHighPrice - previousHighPrice) <= zoneTolerance
        rejection = currentHighPrice - close >= minimumRejection

        if sameZone and rejection
            resistanceLevel = (currentHighPrice + previousHighPrice) * 0.5

            if not f_resistanceExists(resistanceLevel)
                if showSRLines
                    newResistanceLine = line.new(
                         x1=previousHighBar,
                         y1=resistanceLevel,
                         x2=bar_index,
                         y2=resistanceLevel,
                         xloc=xloc.bar_index,
                         extend=extend.right,
                         color=resistanceColor,
                         style=line.style_solid,
                         width=2)

                    array.push(resistanceLines, newResistanceLine)
                    array.push(resistancePrices, resistanceLevel)

                    if array.size(resistanceLines) > maxSRLines
                        oldResistance = array.shift(resistanceLines)
                        array.shift(resistancePrices)
                        line.delete(oldResistance)

            previousHighPrice := currentHighPrice
            previousHighBar := currentHighBar

        else if currentHighPrice > previousHighPrice
            previousHighPrice := currentHighPrice
            previousHighBar := currentHighBar


//---------------------------------------------------------------------
// Support creation
//---------------------------------------------------------------------
if not na(pivotLow)
    currentLowPrice = pivotLow
    currentLowBar = bar_index - pivotRight

    if na(previousLowPrice)
        previousLowPrice := currentLowPrice
        previousLowBar := currentLowBar
    else
        sameZone = math.abs(currentLowPrice - previousLowPrice) <= zoneTolerance
        bounce = close - currentLowPrice >= minimumRejection

        if sameZone and bounce
            supportLevel = (currentLowPrice + previousLowPrice) * 0.5

            if not f_supportExists(supportLevel)
                if showSRLines
                    newSupportLine = line.new(
                         x1=previousLowBar,
                         y1=supportLevel,
                         x2=bar_index,
                         y2=supportLevel,
                         xloc=xloc.bar_index,
                         extend=extend.right,
                         color=supportColor,
                         style=line.style_solid,
                         width=2)

                    array.push(supportLines, newSupportLine)
                    array.push(supportPrices, supportLevel)

                    if array.size(supportLines) > maxSRLines
                        oldSupport = array.shift(supportLines)
                        array.shift(supportPrices)
                        line.delete(oldSupport)

            previousLowPrice := currentLowPrice
            previousLowBar := currentLowBar

        else if currentLowPrice < previousLowPrice
            previousLowPrice := currentLowPrice
            previousLowBar := currentLowBar


//---------------------------------------------------------------------
// Current active support/resistance proximity
//---------------------------------------------------------------------
float nearestSupport = na
float nearestResistance = na

if array.size(supportPrices) > 0
    for i = 0 to array.size(supportPrices) - 1
        level = array.get(supportPrices, i)
        if level <= close
            if na(nearestSupport) or level > nearestSupport
                nearestSupport := level

if array.size(resistancePrices) > 0
    for i = 0 to array.size(resistancePrices) - 1
        level = array.get(resistancePrices, i)
        if level >= close
            if na(nearestResistance) or level < nearestResistance
                nearestResistance := level


//---------------------------------------------------------------------
// S/R reaction conditions
//---------------------------------------------------------------------
nearSupport =
     not na(nearestSupport) and
     math.abs(low - nearestSupport) <= zoneTolerance

nearResistance =
     not na(nearestResistance) and
     math.abs(high - nearestResistance) <= zoneTolerance

supportReaction =
     nearSupport and
     bull and
     close > open and
     close - low >= minimumRejection

resistanceReaction =
     nearResistance and
     bear and
     close < open and
     high - close >= minimumRejection

srBuy =
     timeframeAllowed and
     supportReaction

srSell =
     timeframeAllowed and
     resistanceReaction


//=====================================================================
// 12. FINAL THREE-ENGINE CONFIRMATION
//=====================================================================
buyEngineCount =
     (reversalBuy ? 1 : 0) +
     (specialBuy ? 1 : 0) +
     (srBuy ? 1 : 0)

sellEngineCount =
     (reversalSell ? 1 : 0) +
     (specialSell ? 1 : 0) +
     (srSell ? 1 : 0)

rawBuy = confirmed and buyEngineCount >= 2
rawSell = confirmed and sellEngineCount >= 2


//=====================================================================
// 13. NO REPEATED SAME-DIRECTION SIGNAL
//=====================================================================
var int lastFinalDirection = 0

finalBuy = rawBuy and lastFinalDirection != 1
finalSell = rawSell and lastFinalDirection != -1

if finalBuy
    lastFinalDirection := 1

if finalSell
    lastFinalDirection := -1


//=====================================================================
// 14. FINAL PATTERN NAME
//=====================================================================
string finalPatternName = ""

if finalBuy
    if reversalBuy and reversalPattern != ""
        finalPatternName := reversalPattern
    else if specialBuy and specialPattern != ""
        finalPatternName := specialPattern
    else
        finalPatternName := "منطقة دعم"

if finalSell
    if reversalSell and reversalPattern != ""
        finalPatternName := reversalPattern
    else if specialSell and specialPattern != ""
        finalPatternName := specialPattern
    else
        finalPatternName := "منطقة مقاومة"


//=====================================================================
// 15. FINAL CONFIRMATION STRENGTH
//=====================================================================
finalStrength =
     finalBuy ? buyEngineCount :
     finalSell ? sellEngineCount :
     0

goldConfirmation =
     finalStrength >= 2

pinkConfirmation =
     finalStrength >= 3


//=====================================================================
// 16. SIGNAL BOXES - FINAL VISUAL VERSION
//
// BUY:
//   Integrated upward triangle.
//   Triangle is part of the same green signal body.
//   Arabic word "شراء" is inside the body.
//
// SELL:
//   Integrated downward triangle.
//   Triangle is part of the same red signal body.
//   Arabic word "بيع" is inside the body.
//
// label.style_label_up / label.style_label_down are deliberately used
// so the triangle is physically integrated into the same label body.
// No separate triangle is created.
//=====================================================================

buySignalColor = color.rgb(0, 125, 45)
sellSignalColor = color.rgb(205, 25, 25)
signalTextColor = color.white

if showSignals and finalBuy
    buySignalY = low - atr * 0.16

    label.new(
         x=bar_index,
         y=buySignalY,
         text="شراء",
         xloc=xloc.bar_index,
         yloc=yloc.price,
         style=label.style_label_up,
         color=buySignalColor,
         textcolor=signalTextColor,
         size=size.normal)

if showSignals and finalSell
    sellSignalY = high + atr * 0.16

    label.new(
         x=bar_index,
         y=sellSignalY,
         text="بيع",
         xloc=xloc.bar_index,
         yloc=yloc.price,
         style=label.style_label_down,
         color=sellSignalColor,
         textcolor=signalTextColor,
         size=size.normal)


//=====================================================================
// 17. GOLDEN / PINK CONFIRMATION CIRCLES
//
// IMPORTANT:
// The circles are NOT generated by the existence of a signal alone.
//
// GOLD requires at least TWO engines in the same direction.
// PINK requires THREE engines in the same direction.
//
// BUY:
//   signal body
//   small gap
//   GOLD circle below the signal
//
// SELL:
//   GOLD circle above the signal
//   small gap
//   signal body
//
// The circles never occupy the signal body and never cover the Arabic
// words.
//=====================================================================

goldColor = color.rgb(255, 215, 0)
pinkColor = color.rgb(255, 105, 180)

// The integrated label has its triangle below/above the body.
// These distances keep the confirmation circles outside the label.
goldBuyOffset = atr * 0.58
goldSellOffset = atr * 0.58

pinkBuyOffset = atr * 0.76
pinkSellOffset = atr * 0.76

if showConfirmations and finalBuy and goldConfirmation
    label.new(
         x=bar_index,
         y=low - goldBuyOffset,
         text="",
         xloc=xloc.bar_index,
         yloc=yloc.price,
         style=label.style_circle,
         color=goldColor,
         textcolor=color.white,
         size=size.tiny)

    if pinkConfirmation
        label.new(
             x=bar_index,
             y=low - pinkBuyOffset,
             text="",
             xloc=xloc.bar_index,
             yloc=yloc.price,
             style=label.style_circle,
             color=pinkColor,
             textcolor=color.white,
             size=size.tiny)

if showConfirmations and finalSell and goldConfirmation
    label.new(
         x=bar_index,
         y=high + goldSellOffset,
         text="",
         xloc=xloc.bar_index,
         yloc=yloc.price,
         style=label.style_circle,
         color=goldColor,
         textcolor=color.white,
         size=size.tiny)

    if pinkConfirmation
        label.new(
             x=bar_index,
             y=high + pinkSellOffset,
             text="",
             xloc=xloc.bar_index,
             yloc=yloc.price,
             style=label.style_circle,
             color=pinkColor,
             textcolor=color.white,
             size=size.tiny)


//=====================================================================
// 18. DAILY STATISTICS
//
// A trade is successful when price reaches at least 120 points
// in the direction of the issued signal.
//
// The statistics reset at the beginning of each new day.
//=====================================================================
groupStatistics = "الإحصائيات"

successPoints = input.float(
     120.0,
     "نقاط نجاح الصفقة",
     minval=1.0,
     step=1.0,
     group=groupStatistics)

newDay = ta.change(time("D")) != 0

var int successfulTrades = 0
var int losingTrades = 0

var bool tradeActive = false
var int tradeDirection = 0
var float tradeEntry = na

var string lastTradeText = "-"
var string lastPatternText = "-"

if newDay
    successfulTrades := 0
    losingTrades := 0
    tradeActive := false
    tradeDirection := 0
    tradeEntry := na
    lastTradeText := "-"
    lastPatternText := "-"


//=====================================================================
// 19. POINT CONVERSION
//=====================================================================
pointSize = syminfo.mintick
targetDistance = successPoints * pointSize


//=====================================================================
// 20. START NEW TRADE
//=====================================================================
if finalBuy
    tradeActive := true
    tradeDirection := 1
    tradeEntry := close
    lastTradeText := "صفقة شراء"
    lastPatternText := finalPatternName

if finalSell
    tradeActive := true
    tradeDirection := -1
    tradeEntry := close
    lastTradeText := "صفقة بيع"
    lastPatternText := finalPatternName


//=====================================================================
// 21. TRADE RESULT
//=====================================================================
if tradeActive and not finalBuy and not finalSell
    buyTargetReached =
         tradeDirection == 1 and
         high >= tradeEntry + targetDistance

    sellTargetReached =
         tradeDirection == -1 and
         low <= tradeEntry - targetDistance

    buyFailed =
         tradeDirection == 1 and
         low <= tradeEntry - targetDistance

    sellFailed =
         tradeDirection == -1 and
         high >= tradeEntry + targetDistance

    if buyTargetReached
        successfulTrades += 1
        tradeActive := false

    else if sellTargetReached
        successfulTrades += 1
        tradeActive := false

    else if buyFailed
        losingTrades += 1
        tradeActive := false

    else if sellFailed
        losingTrades += 1
        tradeActive := false


//=====================================================================
// 22. STATISTICS PERCENTAGES
//=====================================================================
totalCompletedTrades = successfulTrades + losingTrades

successRate =
     totalCompletedTrades > 0 ?
     successfulTrades * 100.0 / totalCompletedTrades :
     0.0

lossRate =
     totalCompletedTrades > 0 ?
     losingTrades * 100.0 / totalCompletedTrades :
     0.0


//=====================================================================
// 23. SESSION DETECTION
//
// UTC session windows.
//=====================================================================
utcHour = hour(time, "UTC")

string currentSession = ""

if utcHour >= 0 and utcHour < 3
    currentSession := "جلسة آسيا"
else if utcHour >= 3 and utcHour < 8
    currentSession := "جلسة طوكيو"
else if utcHour >= 8 and utcHour < 13
    currentSession := "جلسة لندن"
else
    currentSession := "جلسة نيويورك"


//=====================================================================
// 24. STATISTICS TABLE
//
// Five vertical cells.
// Upper-right.
// Dark black background.
// Thin white borders.
//=====================================================================
var table statisticsTable = table.new(
     position.top_right,
     1,
     5,
     bgcolor=color.rgb(5, 5, 5),
     frame_color=color.white,
     frame_width=1,
     border_color=color.white,
     border_width=1)

if showStatistics and barstate.islast
    table.cell(
         statisticsTable,
         0,
         0,
         "الصفقات الناجحة\n" + str.tostring(successfulTrades),
         text_color=color.rgb(0, 255, 70),
         bgcolor=color.rgb(5, 5, 5),
         text_size=size.tiny)

    table.cell(
         statisticsTable,
         0,
         1,
         "الصفقات الخاسرة\n" + str.tostring(losingTrades),
         text_color=color.rgb(255, 60, 60),
         bgcolor=color.rgb(5, 5, 5),
         text_size=size.tiny)

    table.cell(
         statisticsTable,
         0,
         2,
         lastTradeText,
         text_color=color.white,
         bgcolor=color.rgb(5, 5, 5),
         text_size=size.tiny)

    table.cell(
         statisticsTable,
         0,
         3,
         lastPatternText,
         text_color=color.white,
         bgcolor=color.rgb(5, 5, 5),
         text_size=size.tiny)

    table.cell(
         statisticsTable,
         0,
         4,
         currentSession,
         text_color=color.white,
         bgcolor=color.rgb(5, 5, 5),
         text_size=size.tiny)


//=====================================================================
// 25. PIVOT VISUAL CONFIRMATION
//
// Small marks only.
// They are not trading signals.
//=====================================================================
plotshape(
     not na(pivotHigh),
     title="قمة مؤكدة",
     style=shape.circle,
     location=location.abovebar,
     color=color.new(resistanceColor, 70),
     size=size.tiny,
     offset=-pivotRight)

plotshape(
     not na(pivotLow),
     title="قاع مؤكد",
     style=shape.circle,
     location=location.belowbar,
     color=color.new(supportColor, 70),
     size=size.tiny,
     offset=-pivotRight)


//=====================================================================
// 26. ALERTS
//=====================================================================
alertcondition(
     finalBuy,
     title="BUY SIGNAL",
     message="XAUUSD PRO: BUY signal - two or more engines confirmed")

alertcondition(
     finalSell,
     title="SELL SIGNAL",
     message="XAUUSD PRO: SELL signal - two or more engines confirmed")

alertcondition(
     finalBuy and pinkConfirmation,
     title="THREE ENGINE BUY",
     message="XAUUSD PRO: BUY confirmed by three engines")

alertcondition(
     finalSell and pinkConfirmation,
     title="THREE ENGINE SELL",
     message="XAUUSD PRO: SELL confirmed by three engines")


//=====================================================================
// END OF XAUUSD PRO SMART SIGNALS
//=====================================================================
