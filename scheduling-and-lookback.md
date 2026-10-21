# Scheduling & the Lookback Buffer (the `-70m@m` note)

A short, practical note on **how often a correlation search runs**, **what time range it covers**,
and **why analysts add a lookback buffer** — the reason a "last 60 minutes" search often uses a
time range like **`-70m@m` to `-10m@m`** instead of `-60m@m` to `now`.

## Splunk relative-time modifiers (quick refresher)
- `-60m` = 60 minutes ago; `-70m` = 70 minutes ago.
- `@m` = **snap to the start of the minute** (drop seconds). `@h` snaps to the hour, `@d` to the day.
- So **`-70m@m`** = "70 minutes ago, snapped to the top of that minute."
- A search's **Earliest** and **Latest** fields use these. Example window: Earliest `-70m@m`,
  Latest `-10m@m` → a **60-minute window** that **ends 10 minutes in the past**.

## Cron scheduling
Scheduled searches use **cron** for *when to run*. Example — run every 60 minutes at 5 past the hour:
```
5 * * * *
```
Cron controls the **trigger time**; the **time-range modifiers** control the **data window**.
The two are set independently, which is exactly why the window can look "shifted" relative to now.

## Why add a lookback buffer?
Events are **not** in the index the instant they occur. There is normal delay from:
- **Ingestion / indexing latency** (forwarders, queues, parsing).
- **Source lag** — many logs are written/forwarded in batches (cloud/API logs especially).
- **Clock skew** between sources and the indexer (events timestamped slightly off).

If a search runs at the top of the hour for `-60m@m` to `now`, any event that happened in the
last window **but arrived late** is **missed** — it indexes *after* the search already ran, and
the next run's window won't cover it either. Result: **detection gaps**.

### The fix: shift the window back (the buffer)
Run the search but look at a window that **ends a few minutes in the past** and optionally
**starts a bit earlier**, so late-arriving events have time to land:

```
Earliest = -70m@m
Latest   = -10m@m
```
- Still a **60-minute** window (no double-counting, no gaps *if* the schedule matches the window).
- The **10-minute tail** gives late events time to be indexed before they're queried.
- Snapping (`@m`) keeps windows aligned minute-to-minute so consecutive runs **tile** cleanly.

> **Match the window to the cadence.** If you run **every 60 minutes**, make the window **60
> minutes** wide. Overlap → duplicate risk contributions (double-scoring); gap → missed events.
> A common safe pattern is *window width = schedule interval*, shifted back by the buffer.

## How the `-70m@m` in a real search reads
`-70m@m` on its own is just the **earliest** bound. Seeing it usually means: *"this search covers
a 60-minute window that is deliberately offset into the past so indexing delay doesn't cause
missed detections."* The companion **latest** bound (often `-10m@m`) is the other half of that
buffer.

## Practical guidance
- Size the buffer to your **worst realistic ingestion delay** (check with
  `index=... | eval lag=_indextime-_time | stats p95(lag)`).
- Keep **window width = schedule interval** to avoid gaps/overlaps.
- Use `@m` (or coarser) snapping so windows are stable and reproducible.
- Document the chosen buffer so the next analyst understands the offset (see
  [`interview-notes.md`](interview-notes.md) for the "why 70 minutes?" explanation).
