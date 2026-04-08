# Thomas1 Strategy - 6-Step Development Guide

This guide breaks down the development of the TradingView Pine Script indicator into six logical steps that build upon each other.

## Step 1: Foundation & HTF Overlay

**Objective:** Set up the basic structure and implement higher timeframe candle overlay

### Key Features Implemented:
- Indicator declaration with proper settings
- Input configuration for HTF timeframe and colors
- HTF data retrieval using `request.security()`
- Hollow HTF candle drawing using box objects
- Proper alignment using `barmerge.lookahead_on`

### Code Highlights:
```pinescript
[htf_open, htf_high, htf_low, htf_close, htf_time] = request.security(
    syminfo.tickerid, htf_timeframe, 
    [open, high, low, close, time], 
    lookahead=barmerge.lookahead_on)
```

### Testing:
- Verify HTF candles align correctly with chart
- Check that historical data displays accurately
- Ensure candles are hollow with proper borders

---

## Step 2: Countdown Timer Implementation

**Objective:** Add real-time countdown display in the middle of HTF candles

### Key Features Implemented:
- Time remaining calculation
- Format display (minutes:seconds)
- Label positioning in candle center
- Current candle detection logic

### Code Highlights:
```pinescript
int time_remaining = htf_bar_duration - time_elapsed
string countdown_text = is_current_candle ? 
    str.tostring(math.round(time_remaining / 60000)) + "m " + 
    str.tostring(math.round((time_remaining % 60000) / 1000)) + "s" : ""
```

### Testing:
- Verify countdown updates in real-time
- Ensure label only appears on current candle
- Check label positioning accuracy

---

## Step 3: Asia Session Logic

**Objective:** Implement session detection and 1st hour high/low tracking

### Key Features Implemented:
- Timezone conversion (America/New_York)
- Asia session start detection (7 PM EST)
- 1st hour (7-8 PM) high/low tracking
- Bold blue lines for 1st hour levels
- 2nd hour open signal (8 PM)

### Code Highlights:
```pinescript
bool is_first_hour = hour(current_time_est) == asia_start_hour and 
                     minute(current_time_est) >= asia_start_minute
asia_first_high := math.max(asia_first_high, high)
asia_first_low := math.min(asia_first_low, low)
```

### Testing:
- Verify session start/end times
- Check high/low tracking accuracy
- Ensure blue lines are bold and visible
- Confirm signal appears at 8 PM EST

---

## Step 4: Session Enhancement & Breakouts

**Objective:** Add breakout detection and 30-minute zone box

### Key Features Implemented:
- High/low breakout alerts
- "High Broken" and "Low Broken" signals
- Dynamic 30-minute zone box (8:30-9 PM)
- Box expansion with price movement

### Code Highlights:
```pinescript
bool high_broken = got_first_hour_data and close > asia_first_high
bool low_broken = got_first_hour_data and close < asia_first_low
bool is_last_30min = hour(current_time_est) == asia_start_hour + 1 and 
                     minute(current_time_est) >= 30
```

### Testing:
- Verify breakout signals trigger correctly
- Check box appears at 8:30 PM EST
- Ensure box expands properly with price
- Test both sides of breakout

---

## Step 5: Market Structure Implementation

**Objective:** Add BOS/CHOCH detection with dashed lines

### Key Features Implemented:
- Pivot point detection (configurable left/right bars)
- BOS (Break of Structure) detection
- CHoCH (Change of Character) detection
- Dashed lines connecting pivot to breakout
- Centered horizontal label positioning

### Code Highlights:
```pinescript
ph = ta.highest(high, left_bars)
pl = ta.lowest(low, right_bars)
bool bull_bos = not na(last_pivot_low) and ta.crossover(high, last_pivot_high)
bool bull_choch = not na(last_pivot_low) and ta.crossunder(low, last_pivot_low)
```

### Testing:
- Verify pivot detection accuracy
- Check BOS/CHOCH trigger conditions
- Ensure dashed lines render properly
- Confirm labels are centered horizontally

---

## Step 6: Refinement & Optimization

**Objective:** Finalize label positioning, colors, and cleanup

### Key Features Implemented:
- Vertical offset for labels (no line overlap)
- Customizable colors for all elements
- Automatic cleanup of old objects
- Final testing and bug fixes
- Comprehensive README documentation

### Code Highlights:
```pinescript
// Labels positioned slightly above/below lines
style=label.style_label_down  // for highs
style=label.style_label_up    // for lows

// Comprehensive README with usage instructions
```

### Testing:
- Verify no label overlaps with dashed lines
- Test all customizable colors
- Ensure no performance issues
- Check clean chart after long use

---

## Integration Notes

### Dependencies Between Steps:
1. Steps 1-2 provide the foundation and timer
2. Step 3 depends on Step 1 (uses HTF time reference)
3. Step 4 extends Step 3 (same session logic)
4. Step 5 is independent but benefits from session context
5. Step 6 refines all previous steps

### Variable Naming Convention:
- `htf_*` - Higher Timeframe related
- `asia_*` - Asia Session related
- `last_pivot_*` - Market Structure related
- `time_*` - Countdown related

### Performance Considerations:
- Minimize object creation in historically fixed areas
- Use `var` for persistent objects (boxes, lines, labels)
- Update existing objects rather than creating new ones
- Clean up old labels to prevent memory bloat

---

## Modification Guide

### To Change Asia Session Hours:
1. Update `asia_start_hour` input in Step 3
2. Adjust timezone in Step 3 if needed
3. Verify 30-minute box timing in Step 4

### To Add More Features:
1. Identify which step your feature belongs to
2. Add inputs in the appropriate step section
3. Implement logic following the step's pattern
4. Test integration with all previous steps

### To Optimize Performance:
1. Review object creation in Step 6
2. Add object limits if needed
3. Test with maximum bars historical data

---

## Reference Implementation

The complete implementation is available in:
- File: `Thomas1_Strategy.pine`
- Repository: https://github.com/gurbpipanalytica-dev/thomas1-strat

Each section in the code is labeled with comments indicating which step it belongs to for easy reference.
