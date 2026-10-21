# Splunk Risk-Based Alerting (RBA) — Notes & Examples

Study and reference notes on **Risk-Based Alerting (RBA)** in Splunk Enterprise Security:
what it is, why SOCs adopt it, how the Risk data model and risk incident rules fit together,
and practical notes on **scheduling and lookback buffers** for correlation searches.

Written as a defensive, lab/learning reference for SOC analysts and detection engineers.

## Contents
| File | Topic |
|---|---|
| [`rba-overview.md`](rba-overview.md) | What RBA is, risk objects, risk scores, risk notables |
| [`example-correlation-search.md`](example-correlation-search.md) | Example searches (with attribution) + RBA annotations |
| [`scheduling-and-lookback.md`](scheduling-and-lookback.md) | Cron, time modifiers, and the **lookback buffer** (the `-70m@m` note) |
| [`interview-notes.md`](interview-notes.md) | SOC interview Q&A — including **"Why 70 minutes?"** |
| [`references.md`](references.md) | Sources, licences and attribution |

## RBA in one paragraph
Instead of firing a separate alert for every suspicious event, RBA **attributes a risk score**
to **risk objects** (users, hosts) for each noteworthy behaviour, writing *risk events* to a
central **Risk index / data model**. A second layer — a **risk incident rule** — fires a single
**risk notable** only when accumulated risk crosses a threshold (e.g. a user triggers several
different MITRE ATT&CK techniques within 24h). This **cuts alert fatigue** and surfaces
**chains of behaviour** rather than isolated events.

## Scope & safety
- Examples are **generic / lab-oriented**; adapt field names, indexes, and thresholds to your
  environment and **test before production**.
- Detection logic adapted from open sources is **attributed** — see
  [`example-correlation-search.md`](example-correlation-search.md) and [`references.md`](references.md).

---
*Compiled by Osman Onur Uyar as a defensive learning reference. Splunk, Splunk Enterprise
Security, and related marks belong to Splunk/Cisco; see [`references.md`](references.md).*
