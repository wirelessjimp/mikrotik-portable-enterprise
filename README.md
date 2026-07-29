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
| MikroTik hEX S | Primary router — VLANs, DHCP, firewall, WireGuard, containers | ~$69 |
| MikroTik mAP | Second device — AP, RADIUS client, remote site | ~$49 |

**Total: Under $120.** Add an Ethernet cable and a USB drive and you're still under $200.

RouterOS v7 is identical across every MikroTik platform — hEX S, L009, RB5009, CCR2004. Same CLI, same WinBox, same feature set. Learn on a $69 device, deploy on anything.

## What You'll Build

By the end of this guide, your MikroTik is:

- A multi-VLAN router with isolated subnets and per-VLAN DHCP
- A firewall enforcing inter-VLAN isolation with management VLAN access
- A WireGuard VPN server accepting site-to-site and road warrior connections
- A RADIUS server (User Manager) for WPA2-Enterprise authentication
- A container host running speed test and network testing tools
- A DNS server with ad blocking
- A portable demo platform you can deploy anywhere

## Guide Structure

Each lab is a standalone module with prerequisites listed at the top. Follow them in order for a complete build, or jump to what you need if you already have a running config.

### Labs 1-6 — Foundation
Initial setup, packages, USB storage, containers, DNS, and backup.

### Labs 7-10 — VLANs and Infrastructure
Single-bridge VLAN filtering, DHCP, port testing, and firewall isolation.

### Labs 11-15 — Security and VPN
RADIUS/WPA2-Enterprise, WireGuard server and clients, User Manager.

### Lab 16 — Second Device (mAP)
Setting up the mAP as a managed AP, trunk endpoint, and travel router.

### Labs 17-27 — Advanced Topics *(coming soon)*
Enterprise switch integration, dual-WAN, ad blocking, cloud management, and more.

## Quick Start

1. Get a MikroTik hEX S (or any RouterOS v7 device)
2. Start at Lab 1
3. Work through the labs in order
4. Break things, restore from backup (Lab 6), try again

## Prerequisites

- A MikroTik device running RouterOS v7
- A laptop with an Ethernet port (or USB-to-Ethernet adapter)
- WinBox (download from mikrotik.com) or a web browser
- An Ethernet cable
- A USB drive (for container storage)

## File Organization

```
labs/
├── labs-01-06.md    # Foundation (Initial config through backup)
├── labs-07-10.md    # VLANs, DHCP, firewall
├── labs-11-15.md    # RADIUS, WireGuard, User Manager
└── lab-16.md        # mAP setup and configuration
```

## About

Written by Jim Palmer (CWNE #304). Born from three years of building portable enterprise test infrastructure for conferences, trade shows, and training — and learning that the best lab is the one you actually have with you.

## License

This guide is provided as-is for educational purposes. Feel free to use it to learn, build, and break things.
