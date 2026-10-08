# Appendix D — Network Diagrams

These diagrams show example network topologies using the MikroTik configuration built in this guide. The first section is the class kit, as the instructor's script builds it. The sections after it are example topologies from the full guide, which builds on a hEX S, so their port tables differ from your kit.

---

## Class Kit (L009 and mAP)

Your kit is one L009 and one mAP. The cables move between two states as you work through the mAP labs. In both, the WireGuard tunnel joins your L009 (`10.255.255.1`) and your mAP (`10.255.255.2`).

```
 STATE 1: THE mAP ON THE 15 cm JUMPER
          (the mAP's management address is 10.10.255.x)

   class switch --- "Router" cable ---> ether1 +---------+
                                               |  L009   | ether8 --- 15 cm jumper ---> ETH1 +--------+
   laptop --------- 1 meter cable ----> ether7 +---------+                                   |  mAP   |
                                                                                             +--------+

 STATE 2: THE mAP ON THE CLASS NETWORK
          (the mAP's management address is 198.51.100.x)

   class switch --- "Router" cable ---> ether1 +---------+
                                               |  L009   |
   laptop --------- 1 meter cable ----> ether7 +---------+

   class switch --- "mAP" cable (PoE) -> ETH1  +--------+
                                               |  mAP   |
                                               +--------+
```

**Your L009 ports**

| Port | Function | Network |
|------|----------|---------|
| ether1 | WAN, to the class switch | `203.0.113.x` from the class router |
| ether2 | VLAN 20 access | `10.10.20.0/24` |
| ether3 | VLAN 30 access | `10.10.30.0/24` |
| ether4 | VLAN 40 access | `10.10.40.0/24` |
| ether5 | VLAN 40 access | `10.10.40.0/24` |
| ether6 | Guest bridge (`br-guest`) | `10.10.50.0/24` |
| ether7 | Backdoor, its own bridge | `192.168.88.0/24`, the router is `192.168.88.1` |
| ether8 | Trunk to the mAP | VLAN 255 untagged (`10.10.255.0/24`), VLANs 20, 30, and 40 tagged. The guest Wi-Fi lab adds VLAN 50 tagged. |

**Your mAP ports and radios**

| Port or radio | Function | Network |
|---------------|----------|---------|
| ETH1 | WAN side, bridge `br-mgmt` | `10.10.255.x` from the L009 on the jumper, `198.51.100.x` on the class network |
| ETH2 | Your laptop, bridge `br-fallback` | `192.168.89.0/24`, the mAP is `192.168.89.1` |
| wlan1 | Fallback Wi-Fi, in `br-fallback` | `Student31-Fallback` |
| wlan2 | Enterprise Wi-Fi, in `br-fallback` | `Student31-EAP` |
| wlan3 | Guest Wi-Fi, in `br-guest`, VLAN 50 over ETH1 | `Student31-Guest`, open |

**Other networks in the kit:** containers on `172.17.0.0/24` (OpenSpeedTest `.2`, iperf3 `.3`, nginx `.4`), the WireGuard tunnel on `10.255.255.0/24`, and the class Wi-Fi on `172.20.26.0/24`. Use your own label in place of `Student31`.

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

### 5-Port Devices (hEX S, hAP), full guide build

| Port | Function | VLAN |
|------|----------|------|
| ether1 | WAN (internet) | — |
| ether2 | VLAN access | 20 |
| ether3 | VLAN access | 30 |
| ether4 | Backdoor access | 255 (untagged) |
| ether5 | Trunk/Expansion | 20,30,40 tagged; 255 untagged |

### 8-Port Devices (L009, RB5009), full guide build

This is the layout the full guide builds. Your class kit's ports are listed at the top of this appendix.

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
*Last Updated: October 2026*
