# Seasonality Heatmap Alerts — MQL4 Script

A MetaTrader 4 script that constructs a **multi-year seasonal price baseline** by filtering historical bars to a configurable month and day-of-month combination, computing the rolling population mean and standard deviation of closing prices across those matched bars, and firing directional alerts when the current price deviates from the seasonal average beyond a configurable standard deviation threshold.

---

## Overview

Seasonality analysis identifies recurring calendar-driven price patterns by aggregating historical data at the same calendar point across multiple years and measuring the current price's position relative to that historical distribution. This script implements a full statistical seasonal baseline: it iterates every available bar, applies `TimeMonth()` and `TimeDay()` filters to isolate the target calendar window, accumulates closing prices into rolling sum and sum-of-squares accumulators for efficient mean and population standard deviation computation, then computes a signed z-score deviation of the current price from the seasonal average. When the deviation exceeds `TrendThreshold` in either direction, a bullish or bearish seasonal trend alert is fired with full statistical context in the message.

---

## Features

- **Calendar-filtered seasonal baseline** — iterates all available bars using `iTime()` → `TimeMonth()` + `TimeDay()` to isolate matching calendar positions across `LookbackYears` years of history
- **Population standard deviation** — computed from sum and sum-of-squares accumulators: `stdDev = sqrt((sumSq / count) − avg²)` — no auxiliary array required
- **Configurable calendar scope** — `Month = 0` and `DayOfMonth = 0` include all months/days respectively; non-zero values filter to specific calendar points for precise seasonal windows
- **Signed z-score deviation** — `deviation = (currentPrice − seasonalAverage) / seasonalStdDev` compared symmetrically against `±TrendThreshold` for bullish/bearish classification
- **Minimum history validation** — `iBars() >= 365 × LookbackYears` guard with a clear log message if insufficient data is available; zero-match guard prevents division by zero on empty filter results
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)
- Alert message includes current price, seasonal average, standard deviation, and deviation in standard deviations for complete statistical transparency

---

## How It Works

1. Every minute, `CalculateSeasonality()` validates bar count (`iBars() >= 365 × LookbackYears`), then iterates all bars to filter by `TimeMonth()` and `TimeDay()` matching
2. Matched bar closes are accumulated into `sum` and `sumSq`; `count` tracks the total matched samples
3. `avg = sum / count` and `stdDev = sqrt((sumSq / count) − avg²)` are computed and returned by reference
4. `currentPrice = iClose(..., 0)` is fetched; `deviation = (currentPrice − avg) / stdDev` is computed
5. Threshold comparisons fire directional alerts:
   - `deviation >= TrendThreshold` → **Strong Seasonal Trend Detected (Bullish)**
   - `deviation <= −TrendThreshold` → **Strong Seasonal Trend Detected (Bearish)**

---

## Input Parameters

| Parameter          | Type            | Default     | Description                                                              |
|--------------------|-----------------|-------------|--------------------------------------------------------------------------|
| `TradeSymbol`      | string          | `EURUSD`    | Symbol for analysis                                                      |
| `Timeframe`        | ENUM_TIMEFRAMES | `PERIOD_D1` | Timeframe for seasonality analysis (daily recommended)                   |
| `LookbackYears`    | int             | `5`         | Number of historical years to include in seasonal baseline               |
| `Month`            | int             | `0`         | Calendar month to filter (0 = all months)                                |
| `DayOfMonth`       | int             | `0`         | Day of month to filter (0 = all days)                                    |
| `TrendThreshold`   | double          | `0.5`       | Minimum standard deviation deviation required to fire a seasonal alert   |
| `EnableAlerts`     | bool            | `true`      | Fire an on-screen/sound alert                                            |
| `EnableEmail`      | bool            | `false`     | Send an email notification                                               |
| `EnablePush`       | bool            | `false`     | Send a mobile push notification                                          |

---

## Alert Message Format

```
Strong Seasonal Trend Detected (Bullish) on EURUSD (Timeframe: PERIOD_D1)
Current Price: 1.08620
Seasonal Average: 1.07890
Std Dev: 0.00540
Deviation: 1.35 SD
```

---

## Installation

1. Copy `Custom_Correlation_Coefficient_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

> **Note:** At least `365 × LookbackYears` bars of history must be loaded in MT4. For `LookbackYears = 5` on `PERIOD_D1`, ensure at least 5 years of daily bars are available in your terminal's history.

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)
- Sufficient historical data loaded (Tools → History Center)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
