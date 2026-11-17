# INC-003 — DNS Troubleshooting Decision Tree

[← INC-003 Case](INC-003-DNS-Resolution.md) · [Service Desk Overview](../README.md) · [IT Support Hub](../../README.md)

**Related incident:** [INC-003 — Corporate DNS Resolution Failure](INC-003-DNS-Resolution.md)  
**Support level:** L2 Helpdesk  
**Status:** Simulated — diagnostic reference; not executed

## Diagnostic Flow

```mermaid
flowchart TD
    A[User reports hostname access failure] --> B[Inspect IP and DNS settings]
    B --> C{Configuration valid?}
    C -->|No| D[Investigate adapter, DHCP, VPN or DNS settings]
    C -->|Yes| E[Query approved DNS resolver]
    E --> F{Expected DNS response?}
    F -->|Yes| G[Investigate application, proxy, TLS or cache]
    F -->|No| H[Compare affected and known-good hostnames]
    H --> I{Internal names only?}
    I -->|Yes| J[Investigate internal zones, VPN DNS, suffixes]
    I -->|No| K[Investigate resolver, forwarding or filtering]
    D --> L[Document, resolve or escalate]
    G --> L
    J --> L
    K --> L
```

## Decision Guide

| Check | Command | Interpretation |
|---|---|---|
| Client configuration | `ipconfig /all` | Review addresses, gateway, DNS server and suffixes |
| Name resolution | `Resolve-DnsName www.microsoft.com` | Check whether the configured resolver returns the expected response |
| Approved resolver comparison | `Resolve-DnsName www.microsoft.com -Server <APPROVED_DNS_IP>` | Compare results only with an authorized resolver |
| DNS service TCP reachability | `Test-NetConnection <APPROVED_DNS_IP> -Port 53` | TCP 53 check only; DNS also commonly uses UDP, so failure is not conclusive |
| Client DNS cache | `ipconfig /displaydns` | Inspect cached results before considering a flush |

Replace placeholder addresses with approved organizational values. Never expose internal hostnames to unauthorized public DNS resolvers.

## Escalation

Escalate when corporate DNS services are unavailable, multiple users are affected, zone or forwarding changes are needed, or security controls may be involved.

## Evidence and Limitations

This is a proposed diagnostic workflow, not a log of executed commands. Record actual outputs and validate any corrective action in an authorized environment before marking a real incident resolved.

## References & Official Documentation

- [Resolve-DnsName — PowerShell reference](https://learn.microsoft.com/en-us/powershell/module/dnsclient/resolve-dnsname)
- [Test-NetConnection — PowerShell reference](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection)
- [DNS overview — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/networking/dns/dns-overview)

*These references support the methodology and technical procedures; they do not independently validate execution, findings, or control effectiveness.*
