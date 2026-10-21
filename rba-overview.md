# RBA Overview

## The problem RBA solves
Traditional SIEM alerting fires a notable for **every** rule match. High-fidelity rules are rare;
most single events are only *mildly* suspicious, so analysts drown in low-value alerts
(**alert fatigue**) and real attacks hide in the noise.

## The RBA idea
Shift from "alert on every event" to "**accumulate risk, then alert on the aggregate**":

1. **Risk rules** (correlation searches) detect individual suspicious behaviours and, instead of
   (or as well as) creating a notable, **assign a risk score** to a **risk object**.
2. Those contributions are written as **risk events** into the **Risk index** (modelled by the
   **Risk data model**).
3. A **risk incident rule** watches accumulated risk and fires a single **risk notable** when a
   threshold is met (e.g. score > N in 24h, or ≥ M distinct ATT&CK tactics for one user).

## Core concepts
| Term | Meaning |
|---|---|
| **Risk object** | The entity risk is attributed to — typically a `user` or `system` (host). |
| **Risk object type** | `user`, `system`, `other`. |
| **Risk score** | Numeric contribution from a behaviour; tune by severity/confidence. |
| **Risk modifier / annotation** | A single risk contribution, often tagged with **MITRE ATT&CK** technique/tactic. |
| **Risk index / data model** | Where risk events are stored and summarised (accelerated). |
| **Risk incident rule** | The aggregation rule that creates the final **risk notable**. |
| **Risk notable** | The alert an analyst actually triages — backed by the contributing events. |

## Why SOCs like it
- **Fewer, richer alerts:** one notable summarising a *chain* of behaviour.
- **Context built in:** the notable shows every contributing event and its ATT&CK mapping.
- **Tunable:** adjust scores and thresholds rather than switching rules on/off.
- **Better detection of "low and slow":** several weak signals on one user add up.

## ATT&CK alignment
Tagging each risk contribution with a **MITRE ATT&CK** technique lets incident rules reason about
**breadth of attack** ("this user hit Initial Access + Execution + Credential Access today"),
which is far stronger than any single signal.

## Typical build order
1. Pick **risk objects** (users, hosts) and ensure fields are normalised (CIM).
2. Convert/author **risk rules** that write risk events with scores + ATT&CK annotations.
3. Accelerate the **Risk data model**.
4. Author **risk incident rules** (thresholds, time windows, distinct-technique counts).
5. **Tune** scores/thresholds against real data to balance signal vs noise.

See [`example-correlation-search.md`](example-correlation-search.md) for concrete searches and
[`scheduling-and-lookback.md`](scheduling-and-lookback.md) for how often to run them.
