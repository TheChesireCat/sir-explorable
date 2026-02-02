# SIR Explorable - Annotated Guide

This README explains how `sir.html` is organized and how the main pieces work.

## Overview
- Single-file HTML app with inline CSS and JS.
- Two simulation modes: deterministic and stochastic.
- SVG-based plotting of S, I, R curves with bands and percentiles.
- Mobile-friendly UI: collapsible parameter panel + drag box.

## File Structure (sir.html)
- `<style>`: All layout, UI, and plot styling.
- `<body>`:
  - Header with title, description, initial values blurb, and a mobile Controls toggle.
  - Controls panel with N, steps, runs, I0, R0, and the all-runs toggle.
  - Two main panels in `.layout`:
    - Left: parameter selector box for beta/mu.
    - Right: SVG plot + legend + help popover.
- `<script>`: Core simulation logic, rendering, and UI events.

## Key UI Elements
- `#paramBox` (SVG): draggable square to set beta (x) and mu (y).
- `#plot` (SVG): main plot with axes, curves, bands, and labels.
- `#controlsPanel`: contains sliders/inputs for N, steps, runs, I0, R0.
- `#controlsToggle`: mobile-only toggle button to show/hide controls.
- `#controlsSummary`: text summary shown when controls are collapsed on mobile.
- `#helpPopover`: help content toggled by the "?" button.
- `#equationTooltip`: hover/tap tooltip explaining the equations.

## State Model (JS)
`state` holds the current model configuration and UI mode:
- `mode`: 'det' or 'stoch'
- `beta`, `mu`: infection and recovery parameters
- `N`, `steps`, `I0`, `R0`: simulation parameters
- `runs`: number of stochastic runs
- `showAllRuns`: toggles faint per-run paths

The UI updates `state` via input handlers and then calls `scheduleRender()`.

## Simulations
### Deterministic
`simulateDeterministic()` runs the standard discrete SIR update:
- S(t+1) = S(t) - beta * I/N * S
- I(t+1) = I(t) + newInf - mu * I
- R(t+1) = R(t) + mu * I

Outputs are arrays `S`, `I`, `R` of length `steps`.

### Stochastic
`simulateStochastic(runCount)` simulates repeated binomial draws per time step.
Key steps:
- For each run:
  - `newInf ~ Bin(S, lambda)` where lambda depends on I and beta.
  - `newRec ~ Bin(I, mu)`
- Aggregates arrays across runs:
  - Mean S/I/R via running average.
  - Stores per-run samples for S/I/R to compute percentiles.
- Percentiles per time step:
  - median, 25th, 75th, 5th, 95th.

Outputs include:
- Means: `meanS`, `meanI`, `meanR`
- Percentiles: `median*`, `p25*`, `p75*`, `p05*`, `p95*`
- Samples: `sSamples`, `iSamples`, `rSamples` (used when showing all runs)

## Rendering Pipeline
- `render()` computes geometry and draws curves in the SVG.
- `buildPath(series, ...)` converts a time series into an SVG path.
- `buildBandFromBounds(upper, lower, ...)` creates a filled envelope between two percentile curves.
- `buildRunLines(samples, ...)` creates faint paths for all runs (S/I/R) when toggled.

Mode-specific behavior in `render()`:
- Deterministic:
  - Draws S/I/R mean curves only.
  - Clears bands/percentile paths and all-runs groups.
- Stochastic:
  - Draws mean curves for S/I/R.
  - Draws median + 25th/75th lines.
  - Draws 5th–95th filled bands for S/I/R.
  - Optionally draws faint run-by-run paths for S/I/R.

## UI Interactions
- `updateModeUI()` toggles visibility of stochastic-only visuals and disables runs controls when in deterministic mode.
- `scheduleRender()` throttles drawing with `requestAnimationFrame`.
- Controls inputs update `state` and re-render.
- The beta/mu selector uses pointer events with `touch-action: none` to prevent mobile scrolling.

## Mobile Behavior
- Controls panel is hidden by default on mobile.
- `controlsToggle` expands/collapses the panel.
- `controlsSummary` shows current values when collapsed.
- Selector panel is a floating drawer on mobile.

## Equation Tooltip
- Small "i" button next to the equation.
- Tooltip text is mode-specific and toggles on tap/click; closes on outside click.

## Where to Start Reading in Code
- State and DOM refs: near the top of `<script>`.
- Simulation functions: `simulateDeterministic`, `simulateStochastic`.
- Rendering: `render()`, `buildPath`, `buildBandFromBounds`.
- UI logic: `updateModeUI`, input listeners, and mobile sync functions.
