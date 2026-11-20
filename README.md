# IT Support, Systems Administration & Cybersecurity Portfolio

**Fefe Mora · IT Support L1/L2 · Systems Administration · Information Security · Cybersecurity**

![IT Support](https://img.shields.io/badge/IT_Support-L1%2FL2-14283F?style=flat-square) ![Systems Administration](https://img.shields.io/badge/Systems-Administration-14283F?style=flat-square) ![Cybersecurity](https://img.shields.io/badge/Cybersecurity-Blue_Team-059669?style=flat-square) ![Evidence](https://img.shields.io/badge/Evidence-Real_%26_Simulated-64748B?style=flat-square)

**[Versión en español / Spanish version](https://github.com/fefemora/it-portfolio-es)**

I work across IT support, systems administration, and security. Much of my experience comes from troubleshooting systems, helping users get back to work, and keeping IT services running. This portfolio brings that work together with hands-on security investigations, technical documentation, and risk assessment.

The projects below show **what I checked, what I found, and what I could verify**. Some are based on real workstation investigations; others are clearly labeled simulations designed to explore technical decisions without claiming production experience.

## Explore by area

| Area | What you'll find |
|---|---|
| [<img src="assets/icons/it-support.svg" width="32" alt="IT Support icon"> **IT Support & Troubleshooting**](IT-Support/README.md) | Real Windows troubleshooting and three L2 service desk scenarios |
| [<img src="assets/icons/cybersecurity.svg" width="32" alt="Cybersecurity icon"> **Cybersecurity**](Cybersecurity/README.md) | Windows event analysis, endpoint security assessment, and controlled change validation |
| [<img src="assets/icons/information-security.svg" width="32" alt="Information Security icon"> **Information Security / GRC**](Information-Security/Enterprise-Risk-Assessment/README.md) | Fictional enterprise risk assessment mapped to NIST CSF 2.0 and CIS Controls |
| [<img src="assets/icons/systems-administration.svg" width="32" alt="Systems Administration icon"> **Systems Administration**](Systems-Administration/README.md) | Windows Server, Active Directory and identity management focus |
| [<img src="assets/icons/networking.svg" width="32" alt="Networking icon"> **Networking**](Networking/README.md) | DNS, VPN, addressing and structured connectivity troubleshooting |
| [<img src="assets/icons/cloud.svg" width="32" alt="Cloud icon"> **Cloud Networking & Security**](Cloud/README.md) | Azure, cloud access controls and hybrid networking development plans |
| [<img src="assets/icons/documentation.svg" width="32" alt="Documentation icon"> **Technical Documentation**](Documentation/README.md) | Incident reports, evidence practices and reusable documentation |

## Featured work

### <img src="assets/icons/it-support.svg" width="29" alt=""> INC-001 · Windows Update troubleshooting

![Windows 11](https://img.shields.io/badge/Windows-11-14283F?style=flat-square) ![PowerShell](https://img.shields.io/badge/PowerShell-Diagnostics-14283F?style=flat-square) ![Real case](https://img.shields.io/badge/REAL_CASE-Resolved-059669?style=flat-square)

[![Verified disk space recovery](IT-Support/Windows-Troubleshooting-Lab/storage-recovery.svg)](IT-Support/Windows-Troubleshooting-Lab/README.md)

Investigated recurring application updates and error `0x80073D02`. After cleanup and restart, free space increased from **18.6 GiB to 63.1 GiB**, and the updates completed successfully. The issue was resolved, although the exact root cause could not be isolated.

[Read the troubleshooting case](IT-Support/Windows-Troubleshooting-Lab/README.md)

### <img src="assets/icons/it-support.svg" width="29" alt=""> L2 service desk · Identity, DNS and VPN

![ITIL](https://img.shields.io/badge/Service_Desk-L2-14283F?style=flat-square) ![Scope](https://img.shields.io/badge/SIMULATED-3_Case_Studies-64748B?style=flat-square)

[![L2 service desk scenarios](IT-Support/Service-Desk-Incident-Management/incident-overview.svg)](IT-Support/Service-Desk-Incident-Management/README.md)

Three fictional incidents cover Microsoft 365 MFA recovery, DNS resolution, and VPN access to internal resources. Each case focuses on diagnostic reasoning, safe escalation, and the checks that would confirm a successful resolution. **The procedures were not executed against customer systems.**

[Browse the L2 incident cases](IT-Support/Service-Desk-Incident-Management/README.md)

### <img src="assets/icons/information-security.svg" width="29" alt=""> Enterprise information security risk assessment

![NIST](https://img.shields.io/badge/NIST-CSF_2.0-14283F?style=flat-square) ![CIS](https://img.shields.io/badge/CIS_Controls-v8.1_IG1-14283F?style=flat-square) ![Simulated](https://img.shields.io/badge/SIMULATED-Risk_Assessment-64748B?style=flat-square)

[![Risk assessment executive dashboard](Information-Security/Enterprise-Risk-Assessment/reports/executive-dashboard.svg)](Information-Security/Enterprise-Risk-Assessment/README.md)

Built a qualitative risk assessment for a **fictional 50-person organization**. It includes an asset inventory, eight illustrative risk scenarios, proposed control mapping, and a 90-day treatment plan. The findings are hypothetical; no real organization or deployed safeguards were assessed.

[Explore the risk assessment](Information-Security/Enterprise-Risk-Assessment/README.md)

### <img src="assets/icons/cybersecurity.svg" width="29" alt=""> Windows Security Event Analysis

![Windows](https://img.shields.io/badge/Windows-Security_Events-14283F?style=flat-square) ![Blue Team](https://img.shields.io/badge/Blue_Team-Read--only_Investigation-059669?style=flat-square)

[![Windows event analysis overview](Cybersecurity/Windows-Security-Event-Analysis/diagrams/security-review-overview.svg)](Cybersecurity/Windows-Security-Event-Analysis/README.md)

Examined Windows Security events, audit-policy settings, and a bounded correlation between **4624** and **4672**. The workstation's Security log held approximately **2.03 days** of events at the time of measurement, and process-creation auditing was disabled. **No malicious activity was confirmed.**

[Review the findings and evidence](Cybersecurity/Windows-Security-Event-Analysis/README.md)

### <img src="assets/icons/cybersecurity.svg" width="29" alt=""> Windows Security Hardening

![Windows](https://img.shields.io/badge/Windows-Security_Baseline-14283F?style=flat-square) ![Assessment](https://img.shields.io/badge/ASSESSMENT-30_Steps_Documented-059669?style=flat-square) ![Follow-up](https://img.shields.io/badge/FOLLOW--UP-Pending-E5A000?style=flat-square)

[![Windows security baseline workflow](Cybersecurity/Windows-Security-Hardening/diagrams/hardening-workflow.svg)](Cybersecurity/Windows-Security-Hardening/README.md)

Completed **28 technical checks and two assessment/closure steps** across Windows Firewall, Defender, auditing, PowerShell logging, BitLocker, and boot protection. A single approved change, **CHG-001**, raised the Security log limit from **20 MiB to 128 MiB**. The new limit was verified; seven-day retention and further hardening remain unverified or pending.

[Open the hardening assessment](Cybersecurity/Windows-Security-Hardening/README.md)

## Technical strengths

| Area | Evidence in this portfolio |
|---|---|
| **Windows & PowerShell** | Event logs, storage investigation, system checks and validation |
| **IT Support / Service Desk L2** | Incident triage, troubleshooting, escalation and user-facing documentation |
| **Microsoft 365 / Identity** | Simulated Entra ID and MFA recovery investigation |
| **Networking** | DNS and VPN diagnostic workflows |
| **Security Operations** | Windows logon/event analysis and endpoint security baselines |
| **Information Security / GRC** | Risk register, control mapping and treatment planning |
| **Communication** | Technical findings, executive summaries and clearly stated limitations |

## What comes next

I'm building on this foundation with **Active Directory and identity labs**, deeper **network diagnostics**, **SIEM monitoring**, and **cloud networking/security**. These are areas of development, not completed case studies.

## How evidence is handled

Every project distinguishes **real observations**, **simulated scenarios**, and **proposed changes**. I document limitations where a test, root cause, or security outcome has not been established. Public examples exclude confidential customer data and credentials.

---

**GitHub:** [github.com/fefemora](https://github.com/fefemora) · **Language:** [Español](https://github.com/fefemora/it-portfolio-es)
