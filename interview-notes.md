# SOC Interview Notes

Short Q&A you can be asked when a correlation search uses a time window like `-70m@m`. The
headline question is **"Why 70 minutes?"** — it tests whether you understand scheduling,
time ranges, and indexing delay.

## Q: "Why 70 minutes?" (why `-70m@m`?)
**A (short):** It's a **60-minute detection window plus a ~10-minute lookback buffer**. The
search runs roughly hourly and covers one hour of data, but the window is **shifted into the
past** so that events which arrived **late** (ingestion/indexing delay, batched sources, clock
skew) are already in the index when the search runs. Without the buffer those late events fall
between runs and are **never alerted on** — a silent detection gap.

**A (expanded):**
- `-70m@m` is the **earliest** bound; it's usually paired with a **latest** bound like `-10m@m`.
- `-70m@m` → `-10m@m` = **60-minute window ending 10 minutes ago**.
- The 10-minute tail = the **buffer** for indexing latency.
- `@m` **snaps to the minute** so consecutive hourly runs **tile** without gaps or overlap.
- Window width (60m) is matched to the schedule interval (hourly) to avoid **double-counting**
  (overlap) or **missed events** (gap).

## Q: What's the difference between the cron schedule and the time range?
**A:** **Cron** decides *when the search fires* (e.g. `5 * * * *` = hourly at :05). The **time
range modifiers** (`earliest`/`latest`) decide *which data it reads*. They're independent —
that's why a search can run "now" but analyse a window that ended minutes ago.

## Q: What does `@m` do, and why snap?
**A:** `@m` snaps the time to the **start of the minute** (drops seconds). Snapping makes windows
**deterministic and aligned**, so run N ends exactly where run N+1 begins — clean tiling, no
fractional-second gaps or overlaps.

## Q: What happens if you *don't* add a buffer?
**A:** Events that occur inside the window but **index after** the search runs are missed. The
next run's window has already moved on, so they're never evaluated → **false negatives** /
detection gaps. This is common with cloud and API logs that forward in batches.

## Q: What's the downside of too large a buffer / overlap?
**A:**
- Too large a **delay** → slower detection (you learn about activity later).
- **Overlapping** windows → the same event is counted **twice**, inflating risk scores and
  causing duplicate notables. Keep **window width = schedule interval**.

## Q: How would you pick the buffer size?
**A:** Measure real ingestion lag and size the buffer above the high percentile:
```spl
index=<your_index> earliest=-24h
| eval lag = _indextime - _time
| stats p50(lag) p95(lag) max(lag)
```
Set the buffer comfortably above **p95** (and re-check periodically, since lag changes with load).

## Q: How does this relate to RBA specifically?
**A:** RBA risk rules run on a schedule and **write risk contributions**. A missed event = a
missing contribution = an incident rule that may **never cross its threshold**. Correct
scheduling/lookback is what makes the accumulated-risk model **reliable**.

---
### One-liner to remember
> **"70 = 60-minute window + 10-minute lookback buffer, snapped to the minute, so late-indexed
> events still get caught and consecutive runs tile cleanly."**

See [`scheduling-and-lookback.md`](scheduling-and-lookback.md) for the full explanation.
