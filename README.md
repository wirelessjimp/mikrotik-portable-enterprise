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

**Total: Under $160.** Add an Ethernet cable and a USB drive and you're still under $200.

To get dual-band Wi-Fi functionality from a Mikrotik device, consider the MikroTik hAP ac2 or hAP ax2. They are larger and cost more than the mAP 2nD, but are dual-band with more horsepower than the mAP 2nd.

RouterOS v7 is identical across every MikroTik platform — hEX S, L009, RB5009, CCR2004. Same CLI, same WinBox, same feature set. Learn on a $69 device, deploy on anything.

## What You'll Build

By the end of this guide, your MikroTik is:

- A multi-VLAN router with isolated subnets and per-VLAN DHCP
- A firewall enforcing inter-VLAN isolation with management VLAN access
- A WireGuard VPN server accepting site-to-site and road warrior connections
- A RADIUS server (User Manager) for WPA2-Enterprise authentication
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

**Labs 11-15 — Security and VPN**
Wireless AP, RADIUS/WPA2-Enterprise, WireGuard server and clients, Cloud and Back to Home.

**Lab 16 — Second Device (mAP)**
Setting up the mAP as a managed AP, trunk endpoint, and travel router.

**Labs 17-19 — Enterprise Integration**
ICX switch integration, RUCKUS AP integration, and AP configuration.

**Labs 20-23 — Tools and Services**
Traffic analysis, TFTP/FTP/graphing, DLNA/SMB media server, and NTP.

**Labs 24-26 — WAN and Guest Access**
WAN sources (USB tethering, cellular, Wi-Fi client mode), dual-WAN failover, and hotspot captive portal.

**Lab 27 — DNS Advanced**
Static entries, DNS over HTTPS, and ad blocking.

**Lab 28 — Administrative Tasks**
Router identity, passwords, software upgrades, user management, scheduled tasks, and logging.

## Quick Start

1. Get a MikroTik hEX S (or any RouterOS v7 device)
2. Start at Lab 1
3. Work through the labs in order
4. Break things, restore from backup (Lab 3), try again

## Prerequisites

- A MikroTik device running RouterOS v7
- A laptop with an Ethernet port (or USB-to-Ethernet adapter)
- WinBox (download from mikrotik.com) or a web browser
- An Ethernet cable
- A USB drive (for container storage)

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
├── lab-13.md    # WireGuard Server
├── lab-14.md    # WireGuard Clients
├── lab-15.md    # Cloud & Back to Home
├── lab-16.md    # Adding Gear (mAP)
├── lab-17.md    # ICX Switch Integration
├── lab-18.md    # RUCKUS AP Integration
├── lab-19.md    # AP Configuration
├── lab-20.md    # Traffic Analysis
├── lab-21.md    # Useful Tools
├── lab-22.md    # Media Center
├── lab-23.md    # Time & NTP
├── lab-24.md    # WAN Sources
├── lab-25.md    # Dual WAN Failover
├── lab-26.md    # Hotspot & Captive Portal
├── lab-27.md    # DNS Advanced
└── lab-28.md    # Administrative Tasks
```

## About

Written by Jim Palmer (CWNE #304). Born from three years of building portable enterprise test infrastructure for conferences, trade shows, and training — and learning that the best lab is the one you actually have with you.

## License

This guide is provided as-is for educational purposes. Feel free to use it to learn, build, and break things.
