# How-To: Investigating Windows Security Events with PowerShell

[Project overview](../README.md) · [Event analysis](event-analysis.md) · [Portfolio home](../../../README.md)

**Status:** Partially executed and verified against the user's console output.  
**Approach:** Read-only queries. No policy changes, event deletion, or simulated attacks.

## Prerequisites

- Windows PowerShell, preferably opened **as administrator** to access the Security log.
- An existing Windows Security event log.
- Do not publish raw usernames, SIDs, IP addresses, hostnames, or event message content without review.

## Step 1 — Inspect the Security log configuration

Run as a **single line** to avoid paste-order issues:

```powershell
Get-WinEvent -ListLog Security | Select-Object LogName, IsEnabled, RecordCount, MaximumSizeInBytes, LogMode
```

**Purpose:** Determine whether the Security log is enabled, how many records it currently holds, its maximum size, and its retention mode.

**Observed result (verified after retry):**

```text
LogName            : Security
IsEnabled          : True
RecordCount        : 33898
MaximumSizeInBytes : 20971520
LogMode            : Circular
```

**Interpretation:** The Security log is enabled, with 33,898 stored records and a configured maximum of **20 MiB** (20,971,520 bytes). `Circular` means older records may be overwritten as the log fills. The oldest and newest retained event timestamps are verified in Step 3.

**Troubleshooting note:** The first attempted paste placed `Select-Object` before `Get-WinEvent`, leaving a trailing pipeline operator and causing `ParserError: EmptyPipeElement`. This was a command-entry error, **not** a Security log failure. The corrected command ran successfully.

## Step 2 — Count selected Security events in the last seven days

```powershell
$ids = 4624,4625,4634,4672,4688,1102

Get-WinEvent -FilterHashtable @{
    LogName   = 'Security'
    Id        = $ids
    StartTime = (Get-Date).AddDays(-7)
} -ErrorAction SilentlyContinue |
    Group-Object Id |
    Sort-Object Name |
    Select-Object Name, Count
```

**Purpose:** Retrieve matching records from the Security log over a rolling seven-day period and group them by event ID.

**Observed output (copied from an actual console run):**

```text
Name Count
---- -----
4624   638
4634    37
4672   614
4688    13
```

**Validation:** Four event IDs returned records. IDs 4625 and 1102 did not appear in the grouped output. This does not establish that they never occurred; auditing, retention, permissions, query scope, and the use of `-ErrorAction SilentlyContinue` may affect what is returned.

**Interpretation:** Event 4624 reports a successful logon, 4634 a logoff, 4672 special privileges assigned to a new logon, and 4688 process creation. Counts are not unique people or evidence of malicious behavior.

## Step 3 — Determine the retained Security log time span

```powershell
$oldest = Get-WinEvent -LogName Security -Oldest -MaxEvents 1
$newest = Get-WinEvent -LogName Security -MaxEvents 1

[PSCustomObject]@{
    OldestEvent  = $oldest.TimeCreated
    NewestEvent  = $newest.TimeCreated
    RetainedDays = [math]::Round(
        ($newest.TimeCreated - $oldest.TimeCreated).TotalDays, 2
    )
}
```

**Purpose:** Find the timestamp of the first and last currently retained Security records, then calculate the elapsed time between them.

**Observed output (real PowerShell console result; local date formatting):**

```text
OldestEvent         NewestEvent         RetainedDays
-----------         -----------         ------------
15-11-2026 20:02:39 17-11-2026 20:40:43         2,03
```

**Interpretation:** Retained records span **approximately 2.03 days**, from November 15 to November 17, 2026. The previous seven-day filtered query could not provide a full seven days of history based on this subsequently observed retained range. The log is configured for circular retention, which **may** overwrite older entries, but this result alone does not prove overwrite occurred or establish uninterrupted logging across the interval.

**Limitations:** The timestamp range is one point-in-time observation. Event counts and retention metadata came from separate command runs; do not assume they share identical collection times.

## Step 4 — Classify a bounded sample of successful logons

**Original attempt:** An unbounded query retrieved every available 4624 event and processed each XML record. It did not return promptly in the user's session. It was replaced with a capped 100-event query; no configuration change was made.

**Successful bounded command:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} -MaxEvents 100 |
ForEach-Object {
    $xml = [xml]$_.ToXml()
    ($xml.Event.EventData.Data |
        Where-Object Name -eq 'LogonType').'#text'
} |
Group-Object |
Sort-Object Count -Descending |
Select-Object Name, Count
```

**Purpose:** Extract and count only the logon type from the 100 most recent event 4624 records. This avoids collecting or displaying usernames, IP addresses, or full event messages.

**Observed result (real console output):**

```text
Name Count
---- -----
5      100
```

**Interpretation:** All 100 sampled successful logon records were **Logon Type 5 (Service)**. Service logons may be routine Windows activity, and this result does **not** establish malicious or unauthorized access.

**Sampling limitation:** This is a cap of 100 **most recent** records, not a random sample and not a full analysis of the 638 records from the earlier seven-day query. The circular log may also continue changing between commands; the 100-record result must not be extrapolated to all logons.

## Step 5 — Investigate event details (planned)

Before proceeding, confirm Step 1 and select a bounded, privacy-safe inspection query. Record only the fields needed for the investigation and redact identifying information before publication.

## Documentation rules

- Label real observations separately from expected output and hypotheses.
- Keep the exact command, purpose, observed output, interpretation, and limitations for each step.
- Do not fabricate screenshots, collection timestamps, alert results, or conclusions.

## References & Official Documentation

- [Microsoft Learn — Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Event 4624](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4624)
- [Microsoft Learn — Event 4634](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4634)
- [Microsoft Learn — Event 4672](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Microsoft Learn — Event 4688](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)
