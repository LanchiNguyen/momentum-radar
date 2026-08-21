# Momentum Radar

A single-page trading tool that watches the chart so you don't have to.

**Live:** https://lanchinguyen.github.io/momentum-radar/

## What it's for

The failure mode it targets: entering during consolidation because you're
bored and hoping, then bleeding premium to theta while price chops sideways.

Everything here encodes one rule — **compression tells you a move is loading,
volume tells you the move is real, and only the combination is an entry.**

## What it shows

- **Squeeze panel** (TTM-style): Bollinger Bands compressing inside Keltner
  Channels. Red dots = coiling, do not enter. Green dot = released, the move
  is starting. Histogram gives direction.
- **RVOL**: this bar's volume vs. its 20-bar average — the fakeout filter.
  Breakouts under 1.0x are usually fake; 1.5x+ is real participation.
- **Volume profile**: POC and 70% value area over the visible range. Price
  runs through thin nodes and stalls at fat ones.
- **VWAP + Stoch RSI**: demoted from triggers to filters, which is how they
  actually work.

## Alerts

Fire on **closed bars only** — no intrabar noise. Enable browser
notifications and leave the tab in the background:

- Squeeze release, with direction and volume confirmation
- Volume spike with range expansion
- VWAP reclaim / loss on volume

## Running it

It's one self-contained HTML file with no build step and no dependencies.
Open the live URL, or download `index.html` and double-click it.

Data comes from Binance's public market-data API — no account or key needed.
"Demo data" runs a synthetic tape if you want to see it work without a feed.

Educational tool. Not financial advice.
