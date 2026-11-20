# <img src="../assets/icons/it-support.svg" width="32" alt="IT Support"> IT Support & Troubleshooting

![Support](https://img.shields.io/badge/Support-L1%2FL2-14283F?style=flat-square) ![Incidents](https://img.shields.io/badge/Incidents-4_documented-14283F?style=flat-square) ![Evidence](https://img.shields.io/badge/Evidence-1_real_%2B_3_simulated-059669?style=flat-square)

[← Portfolio](../README.md) · [Networking](../Networking/README.md) · [Technical Documentation](../Documentation/README.md)

IT support is about narrowing down the problem without making the situation worse. These cases bring together Windows troubleshooting, incident triage, user communication and security-aware escalation.

[![IT Support case map](it-support-navigation.svg)](Service-Desk-Incident-Management/README.md)

## Incident library

| Case | What happened | Type |
|---|---|---|
| **[INC-001 · Windows Update](Windows-Troubleshooting-Lab/README.md)** | Application updates failed; storage cleanup and restart restored normal updates | **Real · Resolved** |
| **[INC-002 · Microsoft 365 MFA](Service-Desk-Incident-Management/incidents/INC-002-M365-Authentication.md)** | User loses access to a registered authentication method after changing phones | **Simulated · Plan documented** |
| **[INC-003 · DNS resolution](Service-Desk-Incident-Management/incidents/INC-003-DNS-Resolution.md)** | Domain names fail to resolve despite apparent IP connectivity | **Simulated · Plan documented** |
| **[INC-004 · VPN connectivity](Service-Desk-Incident-Management/incidents/INC-004-VPN-Connectivity.md)** | VPN shows connected but internal services remain unreachable | **Simulated · Plan documented** |

*INC-001–INC-004 are portfolio identifiers, not tickets from a single real service desk queue.*

## Real troubleshooting evidence

[![INC-001 storage recovery](Windows-Troubleshooting-Lab/storage-recovery.svg)](Windows-Troubleshooting-Lab/README.md)

The Windows investigation recorded error `0x80073D02`, **44.5 GiB** of recovered free disk space, and successful application updates after cleanup and restart. The issue was resolved; its exact cause remains uncertain.

[Read INC-001 and the sanitized evidence](Windows-Troubleshooting-Lab/README.md)

## L2 service desk scenarios

[![Three simulated L2 incidents](Service-Desk-Incident-Management/incident-overview.svg)](Service-Desk-Incident-Management/README.md)

The three L2 cases explore how to handle authentication, DNS and VPN incidents from the first report through investigation and escalation. They describe **proposed diagnostics**, not tests performed on customer systems.

[Browse the L2 case studies](Service-Desk-Incident-Management/README.md)

## Skills in practice

**Windows 11 · PowerShell · Event Viewer · Microsoft 365 / Entra ID · DNS · VPN · ITIL-aligned incident management · Escalation · Validation**

[← Back to the portfolio](../README.md)
