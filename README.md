# Enterprise Networking at Latvian Prices

**A hands-on MikroTik lab guide for Wi-Fi professionals**

Build a portable enterprise test network for under $200. This guide walks you through configuring MikroTik RouterOS to create a multi-VLAN lab environment with DHCP, firewall isolation, WireGuard VPN, RADIUS authentication, containerized services, and more — using hardware you can throw in a backpack.

---

## Who This Is For

Network engineers, Wi-Fi professionals, and anyone who wants to learn enterprise networking concepts on affordable hardware. No prior MikroTik experience required — just a willingness to learn by doing.

## The Hardware

The entire lab runs on two devices:

| Device | Role | Cost |
|--------|------|------|
| MikroTik L009 | Primary router — VLANs, DHCP, firewall, WireGuard, containers | ~$114 |
| MikroTik mAP 2nD | Second device — AP, RADIUS client, remote site | ~$49 |

**Total: Under $170.** Add an Ethernet cable and a USB drive and you're still under $200.

To get dual-band Wi-Fi functionality from a Mikrotik device, consider the MikroTik hAP ac2 or hAP ax2. They are larger and cost more than the mAP 2nD, but are dual-band with more horsepower than the mAP 2nd.

RouterOS v7 is identical across every MikroTik platform — hEX S, L009, RB5009, CCR2004. Same CLI, same WinBox, same feature set. Learn on a $69 device, deploy on anything.

## What You'll Build

By the end of this guide, your MikroTik is:

- A multi-VLAN router with isolated subnets and per-VLAN DHCP
- A firewall enforcing inter-VLAN isolation with management VLAN access
- A WireGuard VPN server accepting site-to-site and road warrior connections
- A RADIUS server (User Manager) for WPA2/WPA3-Enterprise authentication
- A container host running speed test and network testing tools
- A DNS server with ad blocking
- A captive portal for guest access
- A dual-WAN failover router with automatic backup
- A portable demo platform you can deploy anywhere

## Guide Structure

Each lab is a standalone module with prerequisites listed at the top. Follow them in order for a complete build, or jump to what you need if you already have a running config.

**Labs 01-06 — Foundation**
Initial setup, packages, USB storage, containers, MAC access, and backup.

**Labs 07-10 — VLANs and Infrastructure**
Single-bridge VLAN filtering, DHCP, port testing, and firewall isolation.

**Labs 11-12 — Security**
Wireless AP and RADIUS/WPA2-Enterprise with User Manager.

**Lab 13 — mAP Setup**
Setting up the mAP as a managed AP with fallback access and management Wi-Fi.

**Labs 14-16 — VPN**
WireGuard server, WireGuard clients, and Cloud/Back to Home.

**Lab 17 — mAP Advanced**
WireGuard tunnel from mAP to main router, RoMON remote management.

**Lab 18 — Scripting**
RSC files, configuration scripts, and automated deployment.

**Labs 19-20 — Enterprise Integration**
ICX switch integration and RUCKUS AP integration.

**Labs 21-24 — Tools and Services**
Traffic analysis, TFTP/FTP/graphing, DLNA/SMB media server, and NTP.

**Labs 25-27 — WAN and Guest Access**
WAN sources (USB tethering, cellular, Wi-Fi client mode), dual-WAN failover, and hotspot captive portal.

**Lab 28 — DNS Advanced**
Static entries, DNS over HTTPS, and ad blocking.

**Lab 29 — Administrative Tasks**
Router identity, passwords, software upgrades, user management, scheduled tasks, and logging.

> **Note:** Any sub-lab numbered X.9 (e.g., Lab 5.9, Lab 29.9) is optional or reference material. These cover advanced topics, alternative approaches, or background information that isn't required to complete the core build.

## Quick Start

1. Get a MikroTik L009/RB5009/hEX S (or any RouterOS v7 device)
2. Start at Lab 1
3. Work through the labs in order
4. Break things, restore from backup (Lab 3), try again

## Prerequisites

- A MikroTik device running RouterOS v7
- A laptop with an Ethernet port (or USB-to-Ethernet adapter)
- WinBox (download from mikrotik.com) or a web browser
- An Ethernet cable
- A USB drive (for container storage)

## Security Notice

This guide builds lab networks intended to sit behind an existing firewall — not directly exposed to the internet. If you plan to deploy a MikroTik as your primary edge firewall, additional hardening is required beyond what these labs cover.

In particular: **never expose SSH (port 22) to the public internet.** In September 2026, a critical exploit chain called MikroTrick (CVE-2026-67276 + CVE-2026-86060) was used to hijack MikroTik routers with SSH open to the WAN. Always keep RouterOS updated and use WireGuard for remote management instead of opening management ports directly.

- [MikroTik Security Advisory](https://mikrotik.com/supportsec/september-2026-vulnerability/)
- [CERT Polska Advisory](https://cert.pl/en/posts/2026/09/vulnerabilities-in-mikrotik-routeros-actively-exploited/)

## File Organization

```
labs/
├── lab-01.md    # Initial Configuration
├── lab-02.md    # Packages & Extra Features
├── lab-03.md    # Backup & Restore
├── lab-04.md    # USB Storage
├── lab-05.md    # Containers
├── lab-06.md    # MAC Access & Discovery
├── lab-07.md    # Bridges & VLANs
├── lab-08.md    # DHCP Server
├── lab-09.md    # DNS
├── lab-10.md    # Firewall
├── lab-11.md    # Wireless AP
├── lab-12.md    # User Manager & RADIUS
├── lab-13.md    # mAP Setup
├── lab-14.md    # WireGuard Server
├── lab-15.md    # WireGuard Clients
├── lab-16.md    # Cloud & Back to Home
├── lab-17.md    # mAP Advanced (WireGuard & RoMON)
├── lab-18.md    # Scripting & RSC Files
├── lab-19.md    # Enterprise Switch Integration
├── lab-20.md    # Enterprise AP Integration
├── lab-21.md    # Traffic Analysis
├── lab-22.md    # Useful Tools
├── lab-23.md    # Media Center
├── lab-24.md    # Time & NTP
├── lab-25.md    # WAN Sources
├── lab-26.md    # Dual WAN Failover
├── lab-27.md    # Hotspot & Captive Portal
├── lab-28.md    # DNS Advanced
└── lab-29.md    # Administrative Tasks
```
> **Note:** Any sub-lab numbered X.9 (e.g., Lab 7.9, Lab 10.9) is optional or reference material. These cover advanced topics, alternative approaches, or background information that isn't required to complete the core build.

## About

Written by Jim Palmer (CWNE #304). Born from three years of building portable enterprise test infrastructure for conferences, trade shows, and training — and learning that the best lab is the one you actually have with you.

## License

This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/).

You are free to share and adapt this material, provided you:

- **Credit** Jim Palmer (CWNE #304) as the original author
- **Do not** use it for commercial purposes
- **Share** any derivative work under the same license
