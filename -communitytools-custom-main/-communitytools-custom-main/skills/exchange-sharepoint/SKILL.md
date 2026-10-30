---
name: exchange-sharepoint
description: Microsoft Exchange and SharePoint attack techniques — enumeration, ProxyLogon/ProxyShell/ProxyToken exploitation, mailbox access, SharePoint file exfiltration.
---

# Exchange & SharePoint Attacks

Target Microsoft collaboration infrastructure in enterprise environments.

## Techniques

| Target | Vectors |
|--------|---------|
| **Exchange** | Version fingerprint, NTLM info leak, OAB user enum, ProxyLogon (CVE-2021-26855), ProxyShell (CVE-2021-34473), ProxyToken (CVE-2021-33766), EWS mailbox access, GAL dump |
| **SharePoint** | REST API enumeration, site/user/document access, sensitive file search, CSOM access |

## Workflow

1. Fingerprint Exchange/SharePoint version and exposed endpoints
2. Check for unauthenticated vulnerabilities (Proxy* chain)
3. If authenticated: enumerate mailboxes, exfiltrate email, search SharePoint for credentials
4. Pivot: Exchange often runs on high-privilege server; use for lateral movement to AD

## Reference

- `reference/exchange-enum-access.md` — fingerprinting, user enumeration, mailbox access, GAL dump
- `reference/proxylogon-proxyshell-token.md` — CVE-2021-26855, CVE-2021-34473, CVE-2021-33766 exploitation
- `reference/sharepoint-attacks.md` — site/document enumeration, sensitive file search, file download
