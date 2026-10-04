# Appendix D — Network Diagrams

These diagrams show example network topologies using the MikroTik configuration built in this guide.

---

## Basic Lab Setup

The minimum configuration for following this guide.

```
                    Internet
                        │
                        │
                   ┌────┴────┐
                   │  ISP    │
                   │ Router  │
                   └────┬────┘
                        │
                   ┌────┴────┐
                   │ MikroTik│
              ┌────┤  hEX S  ├────┐
              │    │ (main)  │    │
              │    └────┬────┘    │
              │         │         │
         ether4    ether2-3   ether5
       (backdoor)  (VLANs)  (expansion)
              │         │         │
           Laptop    Clients     └─── To: mAP, Switch, or AP
```

**VLANs:**
- VLAN 20: Data (10.10.20.0/24)
- VLAN 30: Voice (10.10.30.0/24)
- VLAN 40: IoT (10.10.40.0/24)
- VLAN 255: Management (10.10.255.0/24)

---

## Full Lab Setup (All Components)

Complete deployment with all devices from this guide.

```
                         Internet
                             │
                             │
                        ┌────┴────┐
                        │  ISP    │
                        │ Router  │
                        └────┬────┘
                             │ ether1 (WAN)
                        ┌────┴────┐
                        │ MikroTik│
                        │  hEX S  │
                        │ (main)  │
                        └─┬──┬──┬─┘
                          │  │  │
            ┌─────────────┘  │  └─────────────┐
            │                │                │
       ether4           ether2-3          ether5
     (backdoor)         (VLANs)        (trunk/expansion)
            │                │                │
         Laptop          Clients              │
                                              │
                    ┌─────────────────────────┴─────────────────────────┐
                    │                                                   │
               ┌────┴────┐                                         ┌────┴────┐
               │  mAP    │                                         │ Switch  │
               │ (Wi-Fi) │                                         │  (ICX)  │
               └────┬────┘                                         └────┬────┘
                    │                                                   │
            ┌───────┴───────┐                               ┌───────────┼───────────┐
            │               │                               │           │           │
       Fallback         Management                     Access Ports    │      ┌────┴────┐
        Wi-Fi             Wi-Fi                        (VLANs)         │      │ RUCKUS  │
    (192.168.89.x)    (10.10.255.x)                                    │      │   AP    │
                                                                       │      └────┬────┘
                                                                       │           │
                                                                  Test Clients   Wi-Fi
                                                                              (Enterprise)
```

---

## Remote Site with WireGuard

mAP deployed at a remote location, tunneling back to home network.

```
    HOME NETWORK                                      REMOTE SITE
    ────────────                                      ───────────
                                     Internet
         ┌────────────┐                 │                ┌────────────┐
         │  MikroTik  │                 │                │   mAP      │
         │   hEX S    │◄────WireGuard───┼────────────────┤  (remote)  │
         │   (main)   │    Tunnel       │                │            │
         └─────┬──────┘                 │                └──────┬─────┘
               │                        │                       │
         Home Network              Hotel/Client            Laptop via
        (10.10.x.x)                  Network              mAP Wi-Fi
                                                        (10.10.255.x)
```

**How it works:**
1. mAP connects to any internet (hotel, client site, cellular)
2. WireGuard tunnel establishes to home router
3. Laptop connects to mAP's management Wi-Fi
4. Full access to home network as if you were there

---

## Portable Demo Kit

Configuration for trade show or demo use.

```
                    ┌─────────────┐
                    │   Cellular  │
                    │   Hotspot   │
                    └──────┬──────┘
                           │ USB or ether1
                    ┌──────┴──────┐
                    │  MikroTik   │
                    │    hEX S    │
                    └──┬──────┬───┘
                       │      │
               ether5  │      │  VLAN 255
            (trunk)    │      │
                       │      │
                ┌──────┴──┐   │
                │ Switch  │   │
                │  (PoE)  │   │
                └──┬───┬──┘   │
                   │   │      │
              ┌────┘   └────┐ │
              │             │ │
         ┌────┴────┐   ┌────┴─┴──┐
         │Enterprise   │  Demo   │
         │   AP    │   │ Laptop  │
         └────┬────┘   └─────────┘
              │
         Demo Wi-Fi
         (SSID for
         booth visitors)
```

**Features:**
- Cellular WAN for connectivity anywhere
- PoE switch powers the AP
- Demo SSID for visitors
- Management access for presenter
- OpenSpeedTest container for speed demos

---

## Production + Lab Setup

MikroTik as your main home router with a separate lab router for testing.

```
        ISP Modem/ONT
        (bridge mode)
              │
              ▼
       ┌──────────────┐
       │ L009 / RB5009│ ◄── Main router (runs the house)
       │ (production) │
       └──────┬───────┘
              │
     ┌────────┼────────┬─────────────┐
     │        │        │             │
     ▼        ▼        ▼             ▼
   Home    Home    Enterprise    ┌────────┐
 Devices   APs      Switch       │ hEX S  │ ◄── Lab router
 (VLANs)                         │ (lab)  │     (isolated VLAN)
                                 └───┬────┘
                                     │
                              ┌──────┴──────┐
                              │             │
                          Lab APs      Lab Devices
                                       (isolated)
```

**How it works:**

1. **Main router (L009/RB5009)** handles all production traffic:
   - ISP connection and NAT
   - Home VLANs (IoT, guest, etc.)
   - Enterprise switch and APs
   - Firewall and security

2. **Lab router (hEX S)** lives on its own VLAN:
   - Gets internet through main router
   - Completely isolated from production
   - Can break things without affecting the house
   - Runs its own VLANs, DHCP, firewall

3. **Benefits:**
   - Test configurations without risk
   - Learn new features safely
   - Keep production stable
   - WireGuard between them for remote lab access

**Key configuration:**

- Main router: Create a "lab" VLAN (e.g., VLAN 99)
- Connect hEX S WAN port to a port on that VLAN
- Lab router gets DHCP from main router
- Lab router NATs its own internal networks

> **This is how the author runs his home network.** The L009 handles production; the hEX S is the lab environment documented in this guide.

---

## Port Assignment Reference

### 5-Port Devices (hEX S, hAP)

| Port | Function | VLAN |
|------|----------|------|
| ether1 | WAN (internet) | — |
| ether2 | VLAN access | 20 |
| ether3 | VLAN access | 30 |
| ether4 | Backdoor access | 255 (untagged) |
| ether5 | Trunk/Expansion | 20,30,40 tagged; 255 untagged |

### 8-Port Devices (L009, RB5009)

| Port | Function | VLAN |
|------|----------|------|
| ether1 | WAN (internet) | — |
| ether2 | VLAN access | 20 |
| ether3 | VLAN access | 20 |
| ether4 | VLAN access | 30 |
| ether5 | VLAN access | 30 |
| ether6 | VLAN access | 40 |
| ether7 | Backdoor access | 255 (untagged) |
| ether8 | Trunk/Expansion | 20,30,40 tagged; 255 untagged |

---

*Document Version: Draft 1.0*
*Last Updated: March 2026*
