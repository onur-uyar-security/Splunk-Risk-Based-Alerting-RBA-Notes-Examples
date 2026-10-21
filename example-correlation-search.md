# Example Correlation Searches (with RBA annotations)

> **Attribution & safety.** The patterns below are **generic, lab-oriented** examples written
> for learning. Where logic is inspired by Splunk's open detection content
> (**Splunk `security_content` / ES Content Update, Apache-2.0**), that is noted. Field names,
> indexes, data models, and thresholds **must** be adapted to your environment, and every search
> **must be tested** before production. See [`references.md`](references.md).

## 1. A risk rule using `tstats` (accelerated, fast)
`tstats` queries accelerated data models and is the preferred way to write performant risk rules.

```spl
| tstats `summariesonly` count
    min(_time) as firstTime max(_time) as lastTime
    from datamodel=Authentication
    where Authentication.action="failure"
    by Authentication.user Authentication.src Authentication.dest
| `drop_dm_object_name("Authentication")`
| where count >= 20
| `security_content_ctime(firstTime)`
| `security_content_ctime(lastTime)`
```
*Pattern/ macros (`summariesonly`, `drop_dm_object_name`, `security_content_ctime`) follow
Splunk `security_content` conventions (Apache-2.0).* Adjust the `count` threshold to your
baseline.

## 2. Turning a detection into a **risk contribution**
In Enterprise Security, a correlation search becomes a risk rule via its **Adaptive Response →
Risk Analysis** action (configured in the UI), which writes a risk event with a **score** and
**risk object**. Conceptually, each match contributes:

```
risk_object = <user or host field>
risk_object_type = user | system
risk_score = <severity/confidence-based number>
+ annotations: MITRE ATT&CK technique/tactic (e.g. T1110 Brute Force)
```

## 3. A **risk incident rule** (aggregate, then alert) using `tstats` on the Risk DM
Fire **one** risk notable when a single object accumulates enough risk across **distinct**
ATT&CK techniques in a window:

```spl
| tstats `summariesonly`
    sum(All_Risk.calculated_risk_score) as risk_score
    count values(All_Risk.annotations.mitre_attack.mitre_tactic_id) as mitre_tactic_id
    dc(All_Risk.annotations.mitre_attack.mitre_technique_id) as mitre_technique_count
    values(All_Risk.annotations.mitre_attack.mitre_technique_id) as technique_ids
    from datamodel=Risk.All_Risk
    by All_Risk.risk_object All_Risk.risk_object_type
| `drop_dm_object_name("All_Risk")`
| where risk_score >= 100 AND mitre_technique_count >= 3
```
*Field names (`All_Risk.calculated_risk_score`, `annotations.mitre_attack.*`) follow the Splunk
ES **Risk** data model; confirm against your ES version.* This says: *alert when an object has
≥100 total risk AND ≥3 distinct ATT&CK techniques.* Tune both thresholds.

## 4. Scheduling these searches
A risk rule typically runs on a schedule over a recent window (e.g. the last ~60 min) — which is
where the **lookback buffer** matters. See
[`scheduling-and-lookback.md`](scheduling-and-lookback.md) for the `-70m@m` pattern and why the
extra time is added.

## Tuning checklist
- [ ] Start with **low scores**; raise as you learn the baseline.
- [ ] Prefer `tstats` + accelerated data models for performance.
- [ ] Annotate every contribution with **ATT&CK** technique/tactic.
- [ ] Set incident-rule thresholds on **total score** *and* **distinct techniques**.
- [ ] Validate with historical data before enabling.
