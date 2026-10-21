# References, Licences & Attribution

Sources for the content in this repository. Splunk documentation and detection content change
across versions — confirm against the docs for **your** Splunk / Enterprise Security release.

## Splunk — Risk-Based Alerting & Enterprise Security
- **Splunk Enterprise Security — Risk-Based Alerting** (official docs):
  https://docs.splunk.com/Documentation/ES
- **Risk Analysis adaptive response / Risk data model** — Splunk ES docs (as above).
- **Splunk Search Reference — Time modifiers** (`earliest`, `latest`, `@` snapping):
  https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/SearchTimeModifiers
- **Splunk — `tstats` command**:
  https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/Tstats
- **Splunk — schedule / cron for saved searches**:
  https://docs.splunk.com/Documentation/Splunk/latest/Report/Schedulereports
- **Splunk RBA community resources** (Splunk `.conf` talks, Splunk Lantern RBA articles).

## Detection content (Apache-2.0)
- **Splunk `security_content` / ES Content Update (ESCU)** — open detection content and the
  macros (`summariesonly`, `drop_dm_object_name`, `security_content_ctime`) referenced in the
  examples: https://github.com/splunk/security_content
  Licence: **Apache License 2.0** — https://www.apache.org/licenses/LICENSE-2.0
  > Example logic inspired by `security_content` conventions is used here under Apache-2.0 for a
  > defensive learning reference. Adapt and test before any production use.

## Framework
- **MITRE ATT&CK** (technique/tactic annotations in RBA): https://attack.mitre.org/
  > © The MITRE Corporation; referenced under MITRE's terms of use.

## Trademarks
Splunk®, Splunk Enterprise Security, and related marks belong to **Splunk Inc. / Cisco**. This
repository is an independent study reference and is **not affiliated with or endorsed by** Splunk.

---
*Compiled by Osman Onur Uyar. All products, standards and content belong to their respective
owners; trademarks are referenced for identification only. Verify every query and setting against
the official documentation for your environment before relying on it.*
