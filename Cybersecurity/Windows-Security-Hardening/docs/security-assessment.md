# Security Baseline — Assessment Register

[Project overview](../README.md) · [Baseline guide](baseline-guide.md) · [Validation plan](validation-plan.md)

**Assessment phase:** Open. **No hardening configuration changes have been applied.**

| Control domain | Objective | Directly observed in this project | Provisional next step | Status |
|---|---|---|---|---|
| Firewall | Confirm native protection across network profiles | None | Run bounded `Get-NetFirewallProfile` query | **Pending evidence** |
| Antivirus | Review on-device malware protection status | None | Inspect selected `Get-MpComputerStatus` fields | **Pending evidence** |
| Security log retention | Ensure enough history for investigations | None in this project; **prior lab:** 20 MiB Circular, ~2.03-day observed span | Define retention target and review feasibility | **Prior evidence / proposed review** |
| Process creation auditing | Improve process activity visibility if justified | None in this project; **prior lab:** `No Auditing` | Evaluate volume, privacy, and test approval | **Prior evidence / proposed review** |
| Identity & access | Review local role exposure in a privacy-safe way | None | Define narrow, redacted query only when needed | **Not assessed** |
| Update posture | Identify missing or stale protection within defined scope | None | Choose supported read-only check | **Not assessed** |
| Platform protections | Review relevant built-in security features | None | Determine OS/build and device support before selecting checks | **Not assessed** |

## Classification rules

- **Verified:** A command and its result were actually executed and supplied for this assessment.
- **Prior evidence:** A verified result from a different, linked investigation; **not** a fresh measurement.
- **Proposed:** A recommendation or validation method; **not implemented**.
- **Unknown:** Information has not yet been collected. Unknown does **not** mean failed.
- **Not applicable:** Requires explicit platform/context justification before use.

## Assessment criteria

Assess each control against a practical desired outcome, vendor references, the device's role, local policy precedence and compatibility. Do not automatically label every difference from an enterprise benchmark as a vulnerability. Record residual risks and evidence limits.

## Related investigation

[Windows Security Event Analysis — Real case](../../Windows-Security-Event-Analysis/README.md) documented the monitoring visibility concerns that motivated this new baseline project. That evidence is attributed to its original date and context.
