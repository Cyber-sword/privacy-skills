---
name: tunneling
description: Internal network pivoting and traffic tunneling — FRP, Chisel, Ligolo-ng, SSH, ReGeorg, DNS tunneling, EarthWorm, Ngrok.
---

# Tunneling & Pivoting

Establish egress channels and route traffic through compromised hosts into segmented internal networks.

## Techniques

| Tool | Transport | Use Case |
|------|-----------|----------|
| Ligolo-ng | TCP | Best general-purpose pivot — kernel TUN interface, full IP routing |
| Chisel | TCP over HTTP | Outbound-only firewall, HTTP allowed |
| FRP | TCP | Persistent SOCKS5 proxy with auth |
| SSH | TCP | When SSH is available on pivot host |
| ReGeorg | HTTP | Webshell already deployed, no binary upload |
| iodine / dnscat2 | DNS | All TCP/HTTP blocked, DNS egress only |
| EarthWorm | TCP | Windows-only pivots, no dependencies |
| Ngrok | TCP/HTTP | Quick external exposure, no server needed |

## Workflow

1. Identify egress restrictions on pivot host (which ports/protocols reach attacker)
2. Select tool matching available transport
3. Start server/listener on attacker machine
4. Deploy agent/client on pivot host
5. Configure proxychains or add routes
6. Verify connectivity to internal target through tunnel

## Reference

- `reference/frp-chisel-ligolo.md` — FRP SOCKS5, Chisel reverse proxy, Ligolo-ng full-network routing
- `reference/ssh-regeorg-ngrok.md` — SSH port forwarding, ReGeorg webshell SOCKS, EarthWorm, Ngrok
- `reference/dns-covert-tunnel.md` — iodine IP-over-DNS, dnscat2 C2-over-DNS, proxychains config
