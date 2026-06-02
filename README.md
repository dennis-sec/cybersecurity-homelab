# Cybersecurity Homelab — Proxmox, pfSense, Suricata, Pi-hole & WireGuard

This is my home cybersecurity lab. I built it to get hands-on with the things you can't really learn from a course alone — segmenting a network, writing IDS rules, breaking and fixing a VPN, and watching an attack light up a detection in real time.

Everything runs on a single Lenovo ThinkCentre M920s (i5-8500, 24 GB RAM) using Proxmox. The lab network is walled off from my home network behind pfSense, monitored by Suricata, filtered by Pi-hole, and reachable remotely over WireGuard. I attack it from an isolated Kali box.

> Private, isolated lab built for learning. No keys, tokens, or live configs are committed — just documentation and screenshots.
>
> Work in progress — I'm building it in phases. Wazuh (SIEM) and VulnHub targets are next; they're marked as roadmap below.

## Why I built it

I wanted a realistic environment to practice both sides — defending a network and attacking it — and to actually *prove* things work instead of assuming they do. Every phase below ends with a verification step: a scan that fires an alert, a ping that gets blocked, a DNS lookup that gets sinkholed.

## How it's laid out

```mermaid
flowchart TD
  INT([Internet]) --> BT["BT Smart Hub 6<br/>192.168.1.254 — home router"]
  BT -->|Powerline Ethernet| PVE["Proxmox host — Lenovo M920s<br/>192.168.1.50"]
  PVE --> PF["pfSense firewall VM<br/>WAN 192.168.1.181 · LAN 10.10.10.1<br/>+ Suricata IDS · WireGuard VPN"]
  PF --> SW["vmbr1 — isolated lab network<br/>10.10.10.0/24"]
  SW --> KALI["Kali Linux<br/>attacker · 10.10.10.100"]
  SW --> PIH["Pi-hole<br/>DNS filtering · 10.10.10.10"]
  SW --> MEDIA["Jellyfin + *arr stack<br/>self-hosted services"]
  PHONE["Phone via cellular"] -. WireGuard VPN — 10.6.210.0/24 .-> PF
```

The home router hands off to pfSense's WAN, and pfSense firewalls and NATs the isolated `10.10.10.0/24` lab (so it's effectively double-NAT). A default-deny rule stops the lab reaching my home network — I confirmed it by pinging my home gateway from Kali and watching it fail 100% while internet access still worked.

## What's running

| Layer | Tech | Address |
|---|---|---|
| Hypervisor | Proxmox VE 9.1 (Debian 13) | 192.168.1.50 |
| Firewall / router / IDS / VPN | pfSense CE 2.8.1 + Suricata + WireGuard | 10.10.10.1 |
| DNS filtering | Pi-hole (Quad9 + DNSSEC) | 10.10.10.10 |
| Attacker | Kali Linux 2026.1 | 10.10.10.100 |
| Self-hosted services | Jellyfin + *arr stack | 10.10.10.x |

## What I actually did (and verified)

### Phase 1 — Proxmox host

Installed Proxmox VE 9.1 on the M920s. Ran into the new deb822 repo format — disabling a repo needs `Enabled: false`, not commenting out the components, which threw "malformed entry" errors until I worked it out. Added an XFCE desktop for local access. I run root auto-login for convenience, which I know widens the local attack surface — I've noted it as a trade-off; hardening is on the roadmap.

### Phase 2 — Network bridges

Created `vmbr1` as the lab switch with no IP on it, so Proxmox doesn't try to route — pfSense handles all routing, NAT, DHCP and DNS. Putting an IP + NAT on the bridge would've created a second router fighting pfSense.

### Phase 3 — pfSense, Suricata, WireGuard (the core)

- **pfSense CE 2.8.1:** WAN via DHCP from my BT hub, LAN `10.10.10.1`, DHCP pool for the lab, default-deny lab → home.
- **Suricata** in IDS (alert-only) mode on the LAN with ETOpen rules plus a few of my own. I wrote SID `1000001` to catch nmap SYN scans, ran `nmap -sS` from Kali against pfSense, and watched my own rule fire.
- **WireGuard** for remote access from my phone, with DuckDNS tracking my changing home IP. The interesting bug: the peer was saved to config but never synced to the live interface — I traced it with `wg show`, `ifconfig` and the filter log over SSH, and fixed it by forcing a re-sync. Also caught an interface left on "None" IPv4 that would have silently broken tunnel routing.

### Phase 4 — Pi-hole

DNS filtering on `10.10.10.10`, Quad9 upstream with DNSSEC. Beyond ad-blocking, this gives me one place to see every domain a lab host talks to — handy for spotting C2-style callbacks. Verified: `doubleclick.net` → `0.0.0.0`, `google.com` resolves normally.

### Phase 6 — Kali attacker

Kali 2026.1 on the lab network with CPU passthrough. It reaches the internet through pfSense but not my home network — exactly the isolation I was after.

## Roadmap

- Wazuh SIEM for centralised logging and correlation
- VulnHub / Metasploitable targets on a separate, isolated attack-range segment
- Full attack → detect → mitigate write-ups
- Hardening: Proxmox 2FA, fail2ban, non-root admin, backups

## Honest limitations

- The Proxmox host itself sits on my home network, not behind pfSense — normal for a single-box lab, but I'd move pfSense to its own hardware to gate the whole house.
- Single-operator setup with root auto-login; proper auth hardening is on the list.

## Screenshots

Per-phase evidence — the Suricata alert, the blocked ping, the Pi-hole dashboard, the nmap scan, the WireGuard flow — is in `/screenshots`.
