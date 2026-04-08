# Thomas1 Strategy - TradingView Indicator

A comprehensive TradingView Pine Script indicator that displays higher timeframe candles, Asia session analysis, and market structure on your chart.

## Overview

This project implements a multi-timeframe trading indicator in Pine Script v5 that overlays higher timeframe candles, tracks Asia session breakout levels, and displays internal market structure (BOS/CHOCH).

## Features

- **Higher Timeframe Candle Overlay**: Displays hollow candles from any timeframe on your current chart
- **Real-time Countdown**: Shows remaining time until HTF candle closes
- **Asia Session Analysis**: 
  - Tracks 1st hour (7-8 PM EST) high/low levels
  - Signals at 2nd hour open (8 PM EST)
  - Alerts when price breaks 1st hour levels
- **30-Minute Zone Box**: Dynamic box that expands during the last 30 minutes of the 2nd hour
- **Market Structure**: Internal BOS (Break of Structure) and CHoCH (Change of Character) with dashed lines
- **Customizable**: All colors, timeframes, and settings can be adjusted

## Installation

1. Copy the `Thomas1_Strategy.pine` file
2. Open TradingView Pine Editor
3. Paste the code
4. Click "Add to Chart"

## Settings

### Higher Timeframe Settings
- HTF Timeframe: Select higher timeframe (default: 60m)
- HTF Bull/Bear Colors: Customize candle colors
- Border Color: Set border color for HTF candles

### Asia Session Settings  
- Asia Start Hour: Set session start hour (default: 19 for 7 PM EST)
- 30m Zone Color: Color for the last 30-minute box
- 1st Hour Level Color: Color for high/low lines (default: blue #5b9cf6)

### Market Structure Settings
- Show Market Structure: Toggle on/off
- Left/Right Bars: Set pivot detection strength
- BOS/CHOCH Colors: Customize line colors

## Usage

1. Add indicator to any chart timeframe
2. Select your desired higher timeframe in settings
3. Configure Asia session start time for your timezone
4. Adjust colors and transparency as needed
5. Monitor breakout signals when price crosses 1st hour levels

## How It Works

### Higher Timeframe Overlay
The indicator uses `request.security()` with `barmerge.lookahead_on` to ensure proper alignment of historical HTF candles with your chart. Each HTF candle is drawn as a hollow box with a countdown timer.

### Asia Session Logic
- **1st Hour (7-8 PM EST)**: Tracks the high and low, draws bold blue lines
- **2nd Hour Signal**: Triggers at 8 PM EST with "2nd Hour Open" label
- **Breakouts**: Monitors price action and alerts when levels are broken
- **30m Zone**: Creates an expanding box from 8:30-9:00 PM EST

### Market Structure
- Uses pivot points to identify potential BOS and CHoCH levels
- Draws dashed lines between pivot and breakout points
- Centers labels horizontally on lines with proper vertical offset

## Development

This indicator was built in six distinct steps:

1. **Foundation & HTF Overlay** - Basic structure and higher timeframe candles
2. **Countdown Timer** - Real-time countdown display
3. **Asia Session Logic** - 1st hour tracking and breakout signals
4. **Session Enhancement** - 30-minute zone box and visual improvements
5. **Market Structure** - BOS/CHOCH implementation with dashed lines
6. **Refinement** - Label positioning, color customization, and final testing

## Notes

- The indicator uses EST (America/New_York) timezone by default
- Adjust the timezone setting if you need different session times
- HTF candles use `barmerge.lookahead_on` for accurate historical alignment
- Labels and boxes are automatically cleaned up to prevent chart clutter

## Changelog

- **v1.0** - Initial release with all core features

## Author

Developed following Thomas's specifications for custom TradingView analysis
