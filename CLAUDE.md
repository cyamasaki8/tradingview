# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A collection of standalone TradingView Pine Script v6 indicators. Each indicator is a single file with no extension (e.g., `tv-atr-label`, `tv-htf-candle-overlay`). There is no build system, package manager, or test runner — code is written here and pasted into TradingView's Pine Editor to run.

## Pine Script Conventions Used Here

**Version**: All scripts use `//@version=6`.

**File structure pattern**:
1. License header + author comment
2. `indicator()` declaration
3. `// ── Inputs ──` section
4. `// ── Calculations ──` or data-fetch section
5. `// ── Plots / Drawings ──` section

**Drawing-heavy indicators** (boxes, lines, labels) follow this pattern:
- Declare `var` arrays at script scope
- Define a `clearDrawings()` function that deletes each object and clears the arrays
- Inside `if barstate.islast`, call `clearDrawings()` then redraw everything fresh each tick

**HTF data via `request.security()`**: Apply the bar offset `[n]` *inside* the `request.security()` call, not outside on the returned series. Applying `[n]` outside steps back `n` chart bars, which on lower timeframes all land in the same HTF candle — not what you want.

```pine
// Correct
[o1, h1, l1, c1] = request.security(syminfo.tickerid, htfTF,
     [open[1], high[1], low[1], close[1]], lookahead=barmerge.lookahead_off)

// Wrong — all chart bars in the same HTF period return identical values shifted
[o1, h1, l1, c1] = request.security(syminfo.tickerid, htfTF,
     [open, high, low, close], lookahead=barmerge.lookahead_off)
o1[1]  // ← this shifts by chart bars, not HTF bars
```

**Pine v6 syntax specifics**:
- Generic arrays: `array.new<box>()`, `array.new<line>()`, `array.new<label>()` (not type-specific constructors)
- Method syntax: `arr.clear()`, `obj.delete()`, `arr.push()`, `arr.get(i)`
- `switch` expressions with `=>` arms, default arm has no label

## Indicator Inventory

| File | Name | Overlay |
|------|------|---------|
| `tv-atr-statusbar` | ATR Status | yes — shows ATR/ATR×2 in the status bar only |
| `tv-atr-label` | ATR Label | yes — floating label at top of chart |
| `tv-bullvsbear-delta` | Volume Bull vs Bear Delta | no — separate pane histogram + info table |
| `tv-htf-candle-overlay` | HTF Candle Offset | yes — draws HTF candles to the right of price |
