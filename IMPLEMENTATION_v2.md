# MT5 EA Implementation Guide - Thomas Asia Pro v2

## Project Overview
Build Thomas_Asia_Pro_v2.mq5 - A professional-grade MT5 Expert Advisor that implements the TradingView "HTF Overlap & Asia Breakout Pro v2" indicator logic.

**Basis:** Thomas_asia_pro_v2.pine (v6, MPL 2.0)
**Target:** MetaTrader 5 Expert Advisor (.mq5)
**Complexity:** 6 implementation steps, ~600 lines
**Time Estimate:** 3-4 hours for experienced MQL5 developer

---

## 📋 Implementation Checklist

### ✅ Step 1: Foundation & HTF Data Retrieval
- [ ] Project initialization
- [ ] Input parameters
- [ ] Global variables
- [ ] CopyRates() setup
- [ ] OnInit() / OnDeinit()
- [ ] Basic OnTick()

### ✅ Step 2: Visual Elements - HTF Candles with Wicks
- [ ] HTF rectangle drawing
- [ ] Upper wick line (trend)
- [ ] Lower wick line (trend)
- [ ] Countdown timer label

### ✅ Step 3: Asia Session - Session Day Tracking (Pro v2)
- [ ] EST timezone conversion
- [ ] Session day key logic
- [ ] 1st hour high/low tracking
- [ ] Blue level lines (width=2)
- [ ] "2nd Hour Open" signal

### ✅ Step 4: Session Enhancements - 30min Zone Box
- [ ] 20:30-21:00 EST detection
- [ ] Dynamic box creation
- [ ] Box expansion logic
- [ ] Breakout detection (high/low)

### ✅ Step 5: Market Structure - BOS/CHOCH with Fractals
- [ ] iFractals() integration
- [ ] BOS/CHOCH detection
- [ ] Trend state tracking (gTrendBull)
- [ ] Dashed lines & offset labels

### ✅ Step 6: Trading Logic & Refinement
- [ ] OrderSend() implementation
- [ ] Position management
- [ ] TP/SL configuration
- [ ] Safety validations
- [ ] Debug logging

---

## 📦 Project Files

**GitHub Repository:** https://github.com/gurbpipanalytica-dev/thomas1-strat

**Files:**
- `/Thomas_asia_pro_v2.pine` - Source indicator (reference)
- `/Thomas_Asia_Pro_v2.mq5` - **Target file to create**
- `/Thomas1_Strategy_EA.mq5` - v1 reference (for comparison)

---

## 🎯 Step 1: Foundation & HTF Data Retrieval

### Files to Create:
```
testbed/
└── Thomas_Asia_Pro_v2_Step1.mq5  (170 lines)
```

### Code Template:
```mql5
// Thomas_Asia_Pro_v2_Step1.mq5
// MT5 EA - Foundation Step

#property copyright "Copyright 2024, Thomas Asia Pro v2"
#property version   "1.00"
#define APP_VERSION "2.0.1"

// === INPUT PARAMETERS ===
input ENUM_TIMEFRAMES InpHTFTimeframe = PERIOD_H1;
input color InpHTFBullColor = C'8,153,129';
input color InpHTFBearColor  = C'242,54,69';
input int InpHTFTransparency = 85;
input double InpLotSize = 0.01;
input int InpSlippage = 3;

// === GLOBALS ===
int gHTFTimeframe;
long gHTFTimeframeMs;
MqlRates gHTFRates[2];
MqlRates gCurrentRates[100];
datetime gLastHTFTime = 0;
var int gLastSessionDay = -1;
var float gFirstHourHigh = 0.0;
var float gFirstHourLow = 0.0;

// === INIT ===
int OnInit() {
    gHTFTimeframe = InpHTFTimeframe;
    gHTFTimeframeMs = PeriodSeconds(gHTFTimeframe) * 1000;
    Print("=== Step 1 Initialized ===");
    return(INIT_SUCCEEDED);
}

void OnDeinit(const int reason) {
    ObjectsDeleteAll(0, "HTF_");
    ObjectsDeleteAll(0, "Asia_");
}

// === TICK ===
void OnTick() {
    CopyRates(NULL, gHTFTimeframe, 0, 2, gHTFRates);
    CopyRates(NULL, PERIOD_CURRENT, 0, 100, gCurrentRates);
    
    if(ArraySize(gHTFRates) < 2) return;
    
    MqlRates currentHTF = gHTFRates[0];
    static datetime lastHTFTime = 0;
    bool isNewHTFBar = (currentHTF.time != lastHTFTime);
    
    ProcessHTFData(currentHTF, isNewHTFBar);
    if(isNewHTFBar) lastHTFTime = currentHTF.time;
    ProcessBasicSession();
}

// === HTF PROCESSING ===
void ProcessHTFData(MqlRates &htfCandle, bool isNewBar) {
    datetime currentTime = TimeCurrent();
    if(isNewBar) {
        string boxName = "HTF_Box_" + (string)TimeToStr(htfCandle.time);
        if(ObjectCreate(0, boxName, OBJ_RECTANGLE, 0, htfCandle.time, htfCandle.high,
                       currentTime, htfCandle.low)) {
            color bgColor = (htfCandle.close >= htfCandle.open) ? InpHTFBullColor : InpHTFBearColor;
            ObjectSetInteger(0, boxName, OBJPROP_BGCOLOR, bgColor);
            ObjectSetInteger(0, boxName, OBJPROP_BACK, true);
        }
    }
}

// === SESSION PROCESSING ===
void ProcessBasicSession() {
    datetime nyTime = GetNYTime();
    int hour = TimeHour(nyTime);
    bool inFirstHour = (hour == 19);
    
    int sessionDay = TimeDay(nyTime) * 100 + TimeMonth(nyTime) * 10000 + TimeYear(nyTime);
    
    if(inFirstHour && sessionDay != gLastSessionDay) {
        gLastSessionDay = sessionDay;
        gFirstHourHigh = high[0];
        gFirstHourLow = low[0];
        Print("Session started: Day=" + (string)sessionDay);
    }
    
    if(inFirstHour) {
        gFirstHourHigh = MathMax(gFirstHourHigh, high[0]);
        gFirstHourLow = MathMin(gFirstHourLow, low[0]);
    }
}

datetime GetNYTime() { return(TimeCurrent() - 5 * 3600); }
// EOF Step 1
```

### Verification Checklist:
- [ ] File created successfully
- [ ] File size ~170-180 lines
- [ ] Last line is `// EOF Step 1`
- [ ] No syntax errors at EOF
- [ ] Compiles in MetaEditor without errors

### Test in MetaEditor:
1. Open MetaEditor (F4 from MT5)
2. File → New → Expert Advisor
3. Paste this code
4. Compile (F7)
5. Should show: `0 error(s), 0 warning(s)`

---

## 🎯 Step 2: Visual Elements - HTF Candles with Wicks

### Add to previous file:

**Update object naming:**
```mql5
string gHTFBoxName = "HTF_Body_";
string gHTFWickHighName = "HTF_WickHigh_";
string gHTFWickLowName = "HTF_WickLow_";
string gCountdownLabelName = "Countdown_";
int gObjectCounter = 0;
```

**Update `ProcessHTFData()`:**
```mql5
void ProcessHTFData(MqlRates &htfCandle, bool isNewBar) {
    datetime currentTime = TimeCurrent();
    string suffix = "_" + (string)TimeToStr(htfCandle.time);
    string boxName = gHTFBoxName + suffix;
    string wickHighName = gHTFWickHighName + suffix;
    string wickLowName = gHTFWickLowName + suffix;
    
    if(isNewBar) {
        gObjectCounter++;
        color bgColor = (htfCandle.close >= htfCandle.open) ? InpHTFBullColor : InpHTFBearColor;
        
        // Body box
        ObjectCreate(0, boxName, OBJ_RECTANGLE, 0, htfCandle.time, htfCandle.high,
                    currentTime, htfCandle.low);
        ObjectSetInteger(0, boxName, OBJPROP_BGCOLOR, bgColor);
        ObjectSetInteger(0, boxName, OBJPROP_BACK, true);
        
        // Upper wick line
        float bodyTop = MathMax(htfCandle.open, htfCandle.close);
        ObjectCreate(0, wickHighName, OBJ_TREND, 0, htfCandle.time, bodyTop,
                    currentTime, htfCandle.high);
        ObjectSetInteger(0, wickHighName, OBJPROP_COLOR, bgColor);
        ObjectSetInteger(0, wickHighName, OBJPROP_STYLE, STYLE_SOLID);
        
        // Lower wick line
        float bodyBot = MathMin(htfCandle.open, htfCandle.close);
        ObjectCreate(0, wickLowName, OBJ_TREND, 0, htfCandle.time, bodyBot,
                    currentTime, htfCandle.low);
        ObjectSetInteger(0, wickLowName, OBJPROP_COLOR, bgColor);
        ObjectSetInteger(0, wickLowName, OBJPROP_STYLE, STYLE_SOLID);
    } else {
        // Update existing objects
        if(ObjectFind(0, boxName) >= 0)
            ObjectSetInteger(0, boxName, OBJPROP_TIME2, currentTime);
        if(ObjectFind(0, wickHighName) >= 0)
            ObjectSetInteger(0, wickHighName, OBJPROP_TIME2, currentTime);
        if(ObjectFind(0, wickLowName) >= 0)
            ObjectSetInteger(0, wickLowName, OBJPROP_TIME2, currentTime);
    }
}
```

---

## 🎯 Step 3: Asia Session - Pro v2 Enhancements

### Add session tracking:
```mql5
// Add to globals
var int gLastSessionDay = -1;
var float gFirstHourHigh = 0.0;
var float gFirstHourLow = 0.0;
var int gFirstHourStartBar = -1;
var bool gSecondHourDrawn = false;

// Add functions
void ProcessSessionPro(datetime nyTime) {
    int hour = TimeHour(nyTime);
    int minute = TimeMinute(nyTime);
    int h2 = (InpAsiaStartHour + 1) % 24;
    
    bool inFirstHour = (hour == InpAsiaStartHour);
    bool isSecondHourStart = (hour == h2 && minute == InpAsiaStartMinute);
    
    // Session day key (robust reset)
    int sessionDay = TimeDay(nyTime) * 100 + TimeMonth(nyTime) * 10000 + TimeYear(nyTime);
    
    if(inFirstHour && sessionDay != gLastSessionDay) {
        gLastSessionDay = sessionDay;
        gFirstHourHigh = high[0];
        gFirstHourLow = low[0];
        gFirstHourStartBar = iBarShift(_Symbol, PERIOD_CURRENT, nyTime, false);
        gSecondHourDrawn = false;
        ObjectsDeleteAll(0, "Asia_");
        Print("=== Asia Session Started === Day=" + (string)sessionDay);
    }
    
    // Track range
    if(inFirstHour && gLastSessionDay == sessionDay) {
        gFirstHourHigh = MathMax(gFirstHourHigh, high[0]);
        gFirstHourLow = MathMin(gFirstHourLow, low[0]);
    }
}
```

---

(Remaining steps 4-6 follow same pattern...)

## 🚀 Next Steps: Implementation Flow

1. **Step 1** → Create `Thomas_Asia_Pro_v2_Step1.mq5` (180 lines)
2. **Step 1** → Test compile in MetaEditor
3. **Step 1** → Commit to GitHub
4. **Step 2** → Add to file (visual elements)
5. **Step 2** → Test compile
6. **Step 2** → Commit
7. **Continue** through steps 3-6

## 📚 Reference Files

**Pine Source Code:**
```
https://github.com/gurbpipanalytica-dev/thomas1-strat/raw/master/Thomas_asia_pro_v2.pine
```

**GitHub Repository:**
```
https://github.com/gurbpipanalytica-dev/thomas1-strat
```

---

## ✅ Verification at Each Step

**After each step:** 1. File created successfully 2. Syntax complete (no truncation) 3. Compiles in MetaEditor (0 errors) 4. File committed to GitHub 5. Parents lines match guide exactly

**Test each step:** ```cpp // Should compile cleanly void OnTest() { MqlRates testRates[10]; CopyRates(NULL, PERIOD_H1, 0, 10, testRates); Print("HTF: " + DoubleToStr(testRates[0].close, _Digits)); }
```

---

**Document Version:** 1.0 **Last Updated:** 2025-04-08 **Next:** Create Step 1 file and verify
