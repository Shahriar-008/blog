---
title: I Set Up NordVPN on My VPS and SSL Broke — Here's the Fix
date: 2026-06-27 14:30:00 +0600
categories: [linux, networking]
tags: [vpn, wireguard, nordvpn, mtu, iptables, nftables, ssl, ssh, sysadmin]
description: Setting up NordVPN with WireGuard on a VPS broke SSL connections and nearly locked me out of SSH. Here's exactly what went wrong and how I fixed it.
---

I run a small VPS in Germany for hosting side projects — Coolify, some Docker containers, a reverse proxy. Everything was fine until I decided to route its traffic through NordVPN.

The VPN connected. NordLynx (their WireGuard implementation) came up clean. And then things started breaking.

SSL connections timed out. Some sites became unreachable. SSH felt fragile. This post is about what caused all of it and how I fixed each piece.

## 1. The Setup

I installed NordVPN directly on the VPS (Ubuntu 24.04) using their CLI tool:

```bash
sh <(curl -sSf https://downloads.nordcdn.com/apps/linux/install.sh)
nordvpn login --token <your-token>
nordvpn set technology nordlynx
nordvpn connect Germany
```

NordLynx is WireGuard under the hood. Faster than OpenVPN, less overhead, newer kernel support. That part worked perfectly — the tunnel came up, public IP changed, traffic routed through the VPN. Done, right?

Not even close.

## 2. Problem 1 — SSL Connections Started Timing Out

The first thing I noticed: sites served over HTTPS from the VPS became painfully slow or refused to connect entirely. External `curl` to the server's domains would hang and eventually time out. HTTP worked fine.

This is a classic MTU problem.

### What's Happening

WireGuard adds its own headers to every packet. If your physical interface has an MTU of 1500 and WireGuard's tunnel interface sits at 1420, a full-size 1500-byte TCP packet doesn't fit through cleanly. The packet gets fragmented — or, worse, silently dropped because the `Don't Fragment` flag is set (common with TLS).

The result: the TCP handshake completes (small packets), but the TLS Client Hello (larger, close to full MTU) never arrives. SSL breaks.

### The Fix — MSS Clamping

Instead of changing MTU everywhere (which breaks other things), you clamp the TCP Maximum Segment Size on forwarded traffic. This tells TCP connections to use smaller packets before they ever hit the tunnel.

With `nftables` (default on Ubuntu 24.04):

```bash
nft add rule ip filter FORWARD tcp flags syn tcp option maxseg size set rt mtu
```

With `iptables` (if you're still on the legacy stack):

```bash
iptables -A FORWARD -p tcp --tcp-flags SYN,RST SYN \
  -j TCPMSS --clamp-mss-to-pmtu
```

This tells every TCP connection going through the FORWARD chain: "Use the path MTU, not the interface MTU." After applying this, SSL worked instantly.

> **Side note:** If you're routing the VPS's own traffic through the VPN (not just forwarding), you also need MSS clamping on the OUTPUT chain:
> ```bash
> nft add rule ip filter OUTPUT tcp flags syn tcp option maxseg size set rt mtu
> ```

To make this survive reboots, save your ruleset:

```bash
# With nftables
nft list ruleset > /etc/nftables.conf
systemctl enable nftables

# Or with netfilter-persistent (iptables)
apt install iptables-persistent
netfilter-persistent save
```

## 3. Problem 2 — I Almost Locked Myself Out of SSH

NordVPN, by default, routes _all_ traffic through the tunnel. That includes your SSH session. If the tunnel goes down mid-session, your connection drops. If the VPN's exit node blocks port 22 return traffic, you're locked out.

This is not hypothetical. I had to be careful.

### The Fix — Exempt SSH from the VPN

The cleanest approach: policy-based routing. Traffic to and from port 22 should bypass the VPN entirely and use the regular internet gateway.

```bash
# Create a separate routing table
echo "200 web" >> /etc/iproute2/rt_tables

# Mark SSH traffic (port 22) with fwmark 1
nft add rule ip mangle OUTPUT tcp sport 22 meta mark set 1
nft add rule ip mangle OUTPUT tcp dport 22 meta mark set 1

# Route marked traffic through the main table, not the VPN
ip rule add fwmark 1 table main priority 100
```

This way, even if the VPN drops, SSH traffic stays on the physical interface. You never lose access.

I also made sure the SSH port was explicitly allowlisted in any firewall rules before connecting the VPN. On a VPS with no physical access, losing SSH means rebuilding the server.

```bash
# With nftables
nft add rule ip filter INPUT tcp dport 22 accept

# With iptables  
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

## 4. Problem 3 — IPv6 Leaked My Real IP

NordLynx only tunnels IPv4 by default. IPv6 traffic — if your VPS has it — goes out directly, completely bypassing the VPN. That means any IPv6-capable site sees your real IP address.

### The Fix — Disable IPv6 (or Tunnel It)

I took the simpler path: disable IPv6 system-wide.

```bash
# /etc/sysctl.d/99-disable-ipv6.conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

Then apply:

```bash
sysctl -p /etc/sysctl.d/99-disable-ipv6.conf
```

Be aware: some applications expect IPv6 and may log warnings. But if your primary goal is a clean VPN tunnel with no leaks, disabling IPv6 is the safest move. The alternative — routing IPv6 through the VPN — requires NordVPN to support IPv6, and at the time of writing, they don't fully.

## 5. Problem 4 — DNS Leaks

Even when traffic routes through the VPN, DNS queries can leak to your VPS provider's default resolver (or Google's, or Cloudflare's — whatever is in `/etc/resolv.conf`).

### The Fix — Pin DNS

```bash
# /etc/systemd/resolved.conf.d/nordvpn.conf
[Resolve]
DNS=1.1.1.1
FallbackDNS=9.9.9.9
Domains=~.
```

The `~.` tells systemd-resolved: route _all_ DNS queries through these servers, no exceptions. Combined with the VPN tunnel, this means your DNS goes through the tunnel to Cloudflare — not to your hosting provider.

## Quick Reference

| Problem | Symptom | Root Cause | Fix |
|---------|---------|------------|-----|
| SSL broken | TLS handshake hangs | MTU black hole (WireGuard overhead) | MSS clamping via nftables/iptables |
| SSH lockout risk | Connection drops if VPN fails | All traffic routed through tunnel | Policy routing — mark SSH, bypass VPN |
| IPv6 leak | Real IP visible to IPv6 sites | NordLynx only tunnels IPv4 | Disable IPv6 in sysctl |
| DNS leak | Queries go to VPS provider | Default resolver bypasses VPN | Pin DNS to 1.1.1.1 in systemd-resolved |

## What I Learned Today

Running a VPN client on a server is not the same as running it on a laptop. On a laptop, if the VPN breaks, you notice and fix it. On a server, a broken VPN can mean lost SSH access, leaked traffic, and silent SSL failures. The four fixes above — MSS clamping, SSH bypass, IPv6 disable, DNS pinning — turn a fragile setup into something I can trust.

And save your firewall rules. `nft list ruleset > /etc/nftables.conf` and `netfilter-persistent save` are the difference between a working server and a mystery after the next reboot.
