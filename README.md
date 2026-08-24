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

**US stocks / ETFs** (SPY, QQQ, NVDA...): needs a free API key from
[twelvedata.com](https://twelvedata.com/register) — email signup, no card.
Paste it in the header's key field; it is stored only in your browser's
localStorage and sent nowhere except Twelve Data itself. Stock charts
refresh by polling every 12 seconds (the free tier has no stream), and
regular-hours bars only.

**Tick-by-tick (optional):** a second free key from
[finnhub.io](https://finnhub.io/register) streams every US-stock trade over a
websocket, so the forming candle ticks live like a real terminal. Paste it in
the "tick key" field. The poll keeps running underneath as the authority on
completed bars and volume.

Both keys are also written into the page URL's hash — bookmark the page after
entering them once and the bookmark carries them to any device.

**Crypto**: keyless. Binance, Binance.US and Coinbase are tried in order,
because binance.com refuses US traffic; the source dropdown can pin one.

"Diagnose" probes each source and names what is blocking it.

Educational tool. Not financial advice.
