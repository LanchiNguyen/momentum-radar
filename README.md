# Momentum Radar

A single-page trading tool that watches the chart so you don't have to.

**Live:** https://lanchinguyen.github.io/momentum-radar/

## What it's for

The failure mode it targets: entering during consolidation because you're
bored and hoping, then bleeding premium to theta while price chops sideways.

Everything here encodes one rule — **compression tells you a move is loading,
volume tells you the move is real, and only the combination is an entry.**

## What the reads are built on

The tape (net drift of the last 6 closes) is **clamped to the current
session** on stock feeds — an overnight gap is not momentum, so the first
bars of a session say "session just opened" instead of flashing a
gap-inflated MOVING. RVOL compares each bar to the average of the **same
time-of-day** on prior days, so the 9:30 rush and the lunch lull are judged
against themselves, not a flat average. Reads of past bars recompute levels
causally (no future pivots — practice mode can't cheat), a read is identical
at any zoom, borderline reads say so, and backtest outcomes stop at the
session close because a day trader is flat overnight.

On a MOVING read, the **"How much is left"** card answers the question a
forecast can't: this move's age and travel against the measured distribution
of every completed run on this same chart (quartiles, with sample sizes), a
fresh / mid-move / past-typical-travel position, and the distance to the next
tested level at the move's current pace — history and arithmetic, each
labeled as what it is. The backtest's **chase test** (entering moves fresh vs
≥4 bars old) measures whether late entries actually score worse on your
chart, and the copy admits it when they don't.

A **state timeline** under the verdict strip shows today's path — when the
chart flipped from quiet to coiling to moving — and each chip clicks through
to that bar's full read. "Read current bar" follows the live bar as data
arrives; ‹ › buttons (or ← →) step bar by bar.

## The design principle

An honest tool for an efficient intraday market cannot predict — the built-in
backtest proved this tool's own forecast layer scored a coin flip, so it was
removed. What remains is a **radar, not an oracle**: it detects events
promptly (alerts), names the current state truthfully, and structures your
discipline (journal). The state line says one of five things: **MOVING**
(with or without volume confirmation), **AT A TESTED WALL**, **COILING**,
**CHOP**, or **QUIET** — descriptions of now, never forecasts.

## What it shows

- **State line** (always visible): the one-glance answer — moving, at a wall,
  coiling, chop, or quiet — with RVOL and the nearest tested levels.
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
- VWAP reclaim / loss on volume (suppressed across the session open)
- Level rejection: the Nth failed attempt at a tested price
- Tested break: a close through a level the market probed 3+ times

If the tab was throttled or the network stalled, alerts catch up over
the bars that closed meanwhile instead of silently skipping them.

## Backtest

The "Does it actually work here?" section replays every bar of history
through the exact scoring and alert code that runs live (no hindsight —
levels and statistics are recomputed causally as the replay advances),
then scores each signal by which of ±1 ATR hit first within 20 bars.
It fetches up to ~5,000 bars for the current symbol/timeframe in one
API call and reports: whether trading WITH the MOVING state beat the
baseline, whether volume confirmation earned its keep, whether the
no-trade states truly carried no direction, and per-alert outcomes —
every number with its sample size, and small buckets refuse to quote.

## Running it

It's one self-contained HTML file with no build step and no dependencies.
Open the live URL, or download `index.html` and double-click it.

**US stocks / ETFs** (SPY, QQQ, NVDA...): needs a free API key from
[twelvedata.com](https://twelvedata.com/register) — email signup, no card.
Paste it in the header's key field; it is stored only in your browser's
localStorage and sent nowhere except Twelve Data itself. Regular-hours bars
only.

**Tick-by-tick (optional):** a second free key from
[finnhub.io](https://finnhub.io/register) streams every US-stock trade over a
websocket, so the forming candle ticks live like a real terminal. Paste it in
the "tick key" field.

**Bigger free limit (optional):** [Alpaca](https://app.alpaca.markets/signup)
allows 200 calls/minute with no daily cap, against Twelve Data's 800/day — it
is the source to switch to when the day's quota is gone. Two caveats worth
knowing before you use it:

- Its free equity feed is **IEX only**, a few percent of the consolidated
  tape. RVOL is unaffected (it compares a feed against its own average), but
  raw volume bars and the volume profile are built from a thin slice. The
  status line says so whenever Alpaca is the source answering.
- **Alpaca keys are account credentials, not a read-only data token.**
  Generate them from a *paper* account.

Alpaca's data host has historically rejected the CORS preflight that a
browser must send with auth headers. If that is still true for you, the chart
will say so and Diagnose will name it; nothing else breaks, and Twelve Data
stays the default.

### Spending the quota

Free stock tiers are measured in hundreds of calls per *day*, so the tool
paces itself rather than polling on a fixed clock:

- **Base cadence fits the budget** — daily-capped sources refresh at a sixth
  of a bar (50s on 5m), so a full session plus loads fits 800/day with room.
  A free Finnhub tick key restores true real-time on top.
- **Tab hidden** — no requests at all; returning to the tab fires one
  immediately.
- **Ticks streaming** — the price is already live, so a request is spent only
  to confirm each closed bar.
- **Nothing changing** — the interval backs off toward 5 minutes, which
  covers nights, weekends and halts without a market calendar.
- A per-minute throttle (Twelve Data allows 8 calls/min) is treated as the
  60-second problem it is — it pauses briefly instead of killing the day.

The status line carries a running `n/800 calls today` count, and hitting a
provider's limit stops the loop with a message naming the limit — rather than
the chart silently going stale. **LIVE means live**: the header clock counts
seconds since the newest bar actually *changed*, not since the last HTTP
response, so a provider replaying stale data cannot look healthy. During
market hours a stalled feed turns the status red; after hours it just says
the market is closed. **Diagnose** answers the real question — it shows the
timestamp of the newest bar every source would serve right now, flags the
stale ones, and offers a one-click reload; it lives in its own panel and
never touches the signal log.

All keys are also written into the page URL's hash — bookmark the page after
entering them once and the bookmark carries them to any device. That makes the
bookmark itself a secret: don't share it, and doubly so with an Alpaca pair in
it.

**Crypto**: keyless. Binance, Binance.US and Coinbase are tried in order,
because binance.com refuses US traffic; the source dropdown can pin one.

"Diagnose" probes each source and names what is blocking it.

Educational tool. Not financial advice.
