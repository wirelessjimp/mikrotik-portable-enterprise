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

## How This Class Works

Before class, your L009 and mAP were built from two script files (RSC files). The bridges, VLANs, DHCP servers, firewall, and the other plumbing are already in place. You spend your time on what a script can't do: certificates, containers, passwords, the tunnel, and Wi-Fi.

**Day 1** is Labs 00 to 05, then Labs 14 and 15. That takes you from a connected router to certificates and HTTPS, three containers, a RADIUS server, and file and media services. **Day 2** is Labs 06 to 09 (the mAP, the tunnel, enterprise Wi-Fi, and RoMON), then your pick from Labs 10 to 13, Lab 16, Lab 17, and the advanced labs. Lab 18 and Appendix A come at the end. Near the end of Day 2 the instructor posts a completed file for each device, and Appendix A shows how to use them if you fall behind.

## What You'll Build

- Your own certificate authority, and signed certificates for HTTPS and RADIUS
- Three containers: OpenSpeedTest, iperf3, and an nginx web server
- A RADIUS server (User Manager) for WPA2-Enterprise Wi-Fi, with PEAP and EAP-TLS
- A WireGuard tunnel from the mAP to the L009
- RoMON management of a device that has no IP address
- MikroTik Cloud and Back to Home
- Wi-Fi as a second internet connection, with failover and a hardware button that switches modes
- TFTP, FTP, SMB, and DLNA file services from a USB drive
- A guest Wi-Fi network with a redirect page

## Guide Structure

**Labs 00-05 — Day 1: the router**
Tools, connecting, the Terminal, certificates and HTTPS, containers, and User Manager.

**Labs 06-09 — The mAP**
Setting up the mAP and the tunnel, enterprise Wi-Fi, RoMON, and client certificates (EAP-TLS).

**Labs 10-11 — Cloud**
MikroTik Cloud and Back to Home.

**Labs 12-13 — Dual WAN**
Wi-Fi as a second internet connection, and the button script.

**Labs 14-16 — Services**
File transfer (TFTP and FTP), the media center (SMB and DLNA), and guest Wi-Fi.

**Labs 17-18 — Wrap-up**
Production readiness, and RSC files.

**Advanced Labs**
Labs of your pick from the full guide.

**Appendix A — If You Fall Behind**
Reset a device and load a completed file.

## Prerequisites

- A laptop with an Ethernet port (or a USB-to-Ethernet adapter)
- WinBox 4 and a web browser. VLC is used in Lab 15.
- Your kit: an L009, a mAP, a USB drive, and Ethernet cables

## Security Notice

This guide builds lab networks intended to sit behind an existing firewall — not directly exposed to the internet. If you plan to deploy a MikroTik as your primary edge firewall, additional hardening is required beyond what these labs cover.

In particular: **never expose SSH (port 22) to the public internet.** In September 2026, a critical exploit chain called MikroTrick (CVE-2026-67276 + CVE-2026-86060) was used to hijack MikroTik routers with SSH open to the WAN. Always keep RouterOS updated and use WireGuard for remote management instead of opening management ports directly.

- [MikroTik Security Advisory](https://mikrotik.com/supportsec/september-2026-vulnerability/)
- [CERT Polska Advisory](https://cert.pl/en/posts/2026/09/vulnerabilities-in-mikrotik-routeros-actively-exploited/)

## File Organization

```
labs/
├── lab-00.md      # Open Your Tools
├── lab-01.md      # Connect Your Router and Laptop
├── lab-02.md      # Meet the Terminal
├── lab-03.md      # Certificates and HTTPS
├── lab-04.md      # Containers
├── lab-05.md      # User Manager (RADIUS Server)
├── lab-06.md      # Set Up the mAP
├── lab-07.md      # mAP Enterprise Wi-Fi (RADIUS Test)
├── lab-08.md      # RoMON (Manage a Device Without an IP Address)
├── lab-09.md      # Client Certificates and EAP-TLS
├── lab-10.md      # MikroTik Cloud
├── lab-11.md      # Back to Home
├── lab-12.md      # Dual WAN (Wi-Fi as a Second Internet Connection)
├── lab-13.md      # The Button Script
├── lab-14.md      # File Transfer
├── lab-15.md      # Media Center
├── lab-16.md      # Guest Wi-Fi (Optional)
├── lab-17.md      # Production Readiness (Draft)
├── lab-18.md      # RSC Files
├── advanced.md    # About the Advanced Labs
├── lab-21.md      # Traffic Analysis
├── lab-22.md      # Useful Tools
├── lab-24.md      # Time & NTP
├── lab-25.md      # WAN Sources
├── lab-28.md      # DNS Advanced
├── lab-29.md      # Administrative Tasks
├── appendix-a.md  # If You Fall Behind
├── appendix-b.md  # Reset Procedures
├── appendix-c.md  # Additional MikroTik Capabilities
├── appendix-d.md  # Network Diagrams
└── appendix-e.md  # Useful Links
```

> **Note:** Labs 19, 20, 23, 26, and 27 from the full guide are still in this folder. They are left out of the table of contents because the class covers their material elsewhere.

## About

Written by Jim Palmer (CWNE #304). Born from three years of building portable enterprise test infrastructure for conferences, trade shows, and training — and learning that the best lab is the one you actually have with you.

## License

This work is licensed under a [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](https://creativecommons.org/licenses/by-nc-sa/4.0/).

You are free to share and adapt this material, provided you:

- **Credit** Jim Palmer (CWNE #304) as the original author
- **Do not** use it for commercial purposes
- **Share** any derivative work under the same license
