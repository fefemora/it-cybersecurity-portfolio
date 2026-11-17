# Enterprise Information Security Risk Assessment

**Framework references:** NIST Cybersecurity Framework (CSF) 2.0 · CIS Controls v8.1, Implementation Group 1 (IG1)  
**Organization:** Emerald Technologies (fictional)  
**Project type:** Portfolio case study — simulated assessment  
**Status:** In progress

## Executive Overview

This project develops a practical, qualitative information security risk assessment for **Emerald Technologies**, a fictional 50-person organization. The assessment focuses on identity and access, Windows endpoints, email threats, backups, and remote access.

The goal is to demonstrate how an IT and cybersecurity professional can identify important assets, assess risk, prioritize security improvements, and communicate actionable recommendations to business stakeholders.

> **Evidence disclaimer:** All organizational details, asset inventories, ratings, and findings are fictional. No production environment was assessed. This is an educational portfolio exercise, not a certification or compliance audit.

## Business Context and Scope

### Assumed environment

- Approximately 50 employees using Windows workstations.
- Microsoft 365 for email, collaboration, and identity services.
- Remote access for selected staff through a corporate VPN.
- Business files stored in approved cloud and/or on-premises services.
- A small internal IT function responsible for user support and administration.

### In scope

- Identity and authentication controls.
- Endpoint configuration, patching, and protection.
- Phishing and email-related threats.
- Business data backup and recovery.
- Remote access and privileged access.

### Out of scope

- Physical penetration testing or offensive security testing.
- Formal legal or regulatory compliance certification.
- Live vulnerability scanning or access to real customer systems.
- Quantitative financial loss modeling.

## Assessment Approach

This case study uses a **qualitative 5 × 5 risk matrix** defined specifically for this project. NIST CSF 2.0 is used to organize cybersecurity outcomes, and CIS Controls v8.1 IG1 is used to identify practical baseline safeguards. Neither framework is presented as prescribing this particular scoring model.

1. Identify business processes, information assets, and asset owners.
2. Describe relevant threats, vulnerabilities, and potential business consequences.
3. Score inherent likelihood and impact on a 1–5 scale.
4. Calculate an initial risk score: **Likelihood × Impact**.
5. Identify existing *assumed* controls and relevant gaps.
6. Map proposed improvements to NIST CSF 2.0 outcomes and applicable CIS IG1 safeguards.
7. Recommend risk treatment, owners, target dates, and validation evidence.
8. Reassess residual risk only when controls and assumptions are sufficiently documented.

### Risk scoring (project-defined)

| Score | Rating | Interpretation |
|---|---|---|
| 1–4 | Low | Monitor and manage through routine controls |
| 5–9 | Moderate | Plan proportionate improvements |
| 10–16 | High | Prioritize treatment and management review |
| 17–25 | Critical | Prompt management attention and risk treatment planning |

**Likelihood scale:** 1 = Rare, 2 = Unlikely, 3 = Possible, 4 = Likely, 5 = Almost certain.  
**Impact scale:** 1 = Negligible, 2 = Minor, 3 = Moderate, 4 = Major, 5 = Severe.

Scores are illustrative and should not be interpreted as measured probabilities or financial losses. Risk acceptance requires an authorized business decision.

## Framework Usage

### NIST CSF 2.0

The six CSF Functions — **Govern, Identify, Protect, Detect, Respond, and Recover** — provide a structure for organizing cybersecurity outcomes and communicating priorities. Relevant Categories and Subcategories will be cited in the control mapping as the assessment develops.

### CIS Controls v8.1 — Implementation Group 1

IG1 is a prioritized baseline of essential cyber hygiene safeguards. The treatment plan will map applicable recommendations to specific IG1 safeguards rather than claiming that implementing IG1 alone guarantees security or compliance.

## Planned Deliverables

| Deliverable | Purpose | Status |
|---|---|---|
| [Asset Inventory](assets/asset-inventory.md) | Identify assets, business value, and ownership | Planned |
| [Risk Register](risk-register/risk-register.md) | Document scenarios and inherent risk scores | Planned |
| [Control Mapping](controls/control-mapping.md) | Map risk treatment to CSF 2.0 and CIS IG1 | Planned |
| [Risk Treatment Plan](treatment/risk-treatment-plan.md) | Assign mitigation actions and validation criteria | Planned |
| [Executive Summary](reports/executive-summary.md) | Communicate priorities to management | Planned |

## Intended Skills Demonstrated

- Information security risk assessment
- Business impact and likelihood analysis
- Asset identification and ownership
- NIST CSF 2.0 outcome mapping
- CIS Controls v8.1 IG1 safeguard selection
- Risk treatment prioritization
- Security documentation and executive communication

## References

- [NIST Cybersecurity Framework 2.0](https://www.nist.gov/cyberframework)
- [CIS Critical Security Controls](https://www.cisecurity.org/controls)

---

*This repository documents a fictional assessment for learning and portfolio demonstration. It does not claim an actual organizational assessment, implemented controls, or verified compliance.*
