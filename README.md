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

## Ask — a coach over the numbers, not an oracle

The **Ask** tab sends a question to Claude together with a snapshot of what
the tool has *measured*: the current state and levels, today's state path,
the move-run statistics, alerts, your journal, and the last backtest summary.
It never receives raw candles, and its instructions forbid forecasts —
direction, targets, timing — for the same reason the tool makes none. Where
it earns its keep: **patterns in your own journal** (state at entry, time of
day, failure tags, the language of your theses) and plain-English
explanations of what the panels are showing.

It uses **your own Anthropic API key** (create one at console.anthropic.com;
needs prepaid credits, and set a monthly spend cap there). Calls go straight
from your browser to `api.anthropic.com` — there is no server in between — and
each answer shows its estimated cost, including cache writes, and a running
session total. The key is stored in this browser only and is **never** put in
a link, not even the setup link. Conversation history lives in the tab for the
session; server-side refusal fallbacks are enabled by default.

## The design principle

An honest tool for an efficient intraday market cannot predict — the built-in
backtest proved this tool's own forecast layer scored a coin flip, so it was
removed. What remains is a **radar, not an oracle**: it detects events
promptly (alerts), names the current state truthfully, and structures your
discipline (journal). The state line says one of six things: **BREAKING** (the first close out of
a compression box, on volume — the earliest bar the rule recognises),
**MOVING** (with or without volume confirmation), **AT A TESTED WALL**,
**COILING**, **CHOP**, or **QUIET** — descriptions of now, never forecasts.

The tape threshold that separates MOVING from QUIET is **calibrated to the
chart in front of you**: it is a percentile of that chart's own six-bar moves,
so it fires on a known slice of bars and the read tells you what that slice is.
A fixed threshold fired on more than half the bars of a pure random walk, which
made the biggest word on the screen a description of noise.

## What it shows

- **The state block** (always visible, biggest thing on the page): the
  one-glance answer, in the state's colour only when volume backs it, with a
  confirmation flag, one sentence of why, today's path as a dot strip, and four
  tiles — **the rule** leg by leg (compression, volume, direction, each with its
  measured value and a pass mark), the move's age against the measured
  distribution of finished swings, the next tested wall in dollars and ATR, and
  how much session is left against the median swing length.
- **The chart fills the rest of the screen.** Squeeze, volume and Stoch RSI
  panels are opt-in toggles at the chart's top-right. Read, Alerts and Ask sit
  beside the chart on a wide screen and in a bottom sheet on a narrow one —
  either way the canvas resizes so the bar you clicked stays visible. The
  journal, backtest and learn material live in a separate Review view.
- **Session separators and dates** on the time axis, VWAP breaking at each
  session reset, level lines carrying their price, and markers for the bar the
  verdict speaks for and the bar the current move began on. Reading a past bar
  draws the walls that existed *at that bar*, not today's.
- **Squeeze panel** (TTM-style): Bollinger Bands compressing inside Keltner
  Channels; amber dots while compression is on, the histogram in the up/down
  colours for direction. The rule tile says the same thing in words.
- **RVOL**: this bar's volume against what the symbol normally does **at this
  time of day** (the same clock slot on prior sessions; the last 20 bars until
  three prior days exist). For a move already running, the tool compares the
  volume of the whole move against that baseline, so one quiet bar cannot flip
  the verdict.
- **Volume profile**: POC and 70% value area. Price runs through thin nodes and
  stalls at fat ones.
- **VWAP + Stoch RSI**: demoted from triggers to filters, which is how they
  actually work.

## Alerts

Fire on **closed bars only** — no intrabar noise:

- Compression that has been on five bars — said *before* it releases, since
  that is the setup this whole tool is built around
- Squeeze release, with direction and volume confirmation
- Volume spike with range expansion
- VWAP reclaim / loss on volume (suppressed across the session open)
- Level rejection: the Nth failed attempt at a tested price
- Tested break: a close through a level probed 3+ times, **on volume** —
  structure without volume is not an entry by this tool's own rule
- State changes, including the moment an unconfirmed move becomes
  volume-confirmed
- A move reaching the median length of finished swings on this chart

Alerts only reach you while the tab is open; a backgrounded tab stops polling
to protect the free quota, and catches up on the bars that closed meanwhile
when you return.

If the tab was throttled or the network stalled, alerts catch up over
the bars that closed meanwhile instead of silently skipping them.

## Backtest

The "Does it actually work here?" section replays every bar of history through
the exact scoring and alert code that runs live (no hindsight — levels and
statistics are recomputed causally as the replay advances), then scores each
entry by which of ±1 ATR hit first within 20 bars. A trade that reaches
neither by the session close is marked to that close rather than counted a
loss, because buckets differ in how often that happens and the difference is
not an edge.

**Every edge carries a 90% band** from resampling whole trading sessions, and a
row is coloured only when its band excludes zero. Adjacent bars share almost
all of their outcome window, so the report counts **episodes** — contiguous
stretches, one trade each — alongside bars. On driftless random walks, tapes
with no edge in them by construction, it prints no confident verdict at all;
the previous version told causal stories about them.

It reports the tool's actual claim — all three legs of the rule passing,
compression without volume, volume without compression — plus the MOVING
buckets, the no-trade states, and per-alert outcomes, each with its band.

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

**Keys live in this browser's localStorage and nowhere else.** They used to be
written into the page URL on every load, which put an Alpaca pair — an *account*
credential, not a read-only data token — into browser history, synced history
and every screenshot. To carry them to another device, the ⚙ row has a **copy
setup link** button that builds that link on demand and asks separately before
including the Alpaca secret. Opening such a link stores the keys and then wipes
them from the address bar. The Anthropic key is never in a link at all.

Stock feeds are filtered to **regular hours, 9:30–16:00 ET**. Alpaca serves the
whole extended session, and pre-market bars would anchor VWAP at 4am, fill the
volume baseline with overnight prints and make the 9:30 bar read as a
compression.

**Crypto**: keyless. Binance, Binance.US and Coinbase are tried in order,
because binance.com refuses US traffic; the source dropdown can pin one.

"Diagnose" probes each source and names what is blocking it.

Educational tool. Not financial advice.
