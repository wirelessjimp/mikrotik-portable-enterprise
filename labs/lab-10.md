# Lab 10 — Firewall Configurations

*Prerequisites: Lab 7, Lab 8*

This lab creates firewall rules that isolate VLANs from each other while allowing internet access. Each VLAN can reach the internet but cannot communicate with other VLANs — keeping lab networks isolated.

---

## Lab 10.1 — Find Your WAN Gateway (Quad-Zero Route)

Before creating firewall rules, you need to know your upstream gateway — the IP address of your home router or upstream network device. We find this by looking at the **quad-zero route** (0.0.0.0/0), which is your default route for traffic with no more specific destination.

1. Open a Terminal

2. Run:

    ```
    /ip/route/print where dst-address=0.0.0.0/0
    ```

3. Look at the **gateway** column. This is your WAN gateway IP address.

   Example output:
   ```
   Flags: D - DYNAMIC
   Columns: DST-ADDRESS, GATEWAY, DISTANCE
       DST-ADDRESS     GATEWAY        DISTANCE
   D   0.0.0.0/0       192.168.1.1    1
   ```

   In this example, the WAN gateway is **192.168.1.1**.

4. Record this value on your lab notes page — you'll use it in the firewall rules below.

---

## Lab 10.2 — Understanding the Goal

We want:
1. ✅ VLAN devices can reach the internet
2. ✅ VLAN devices can communicate with other devices on the same VLAN
3. ✅ VLAN devices can reach their gateway (for DHCP, DNS)
4. ✅ VLAN devices can reach containers (172.17.0.x)
5. ❌ VLAN devices cannot reach other VLANs (10.10.20.x cannot reach 10.10.30.x)

VLAN 255 is different — as the management VLAN, it **can** reach all internal networks.

---

## Lab 10.3 — Rule Logic

Each lab VLAN (20, 30, 40) needs four rules:

| Order | Chain | Src Address | Dst Address | In Interface | Action | Purpose |
|-------|-------|-------------|-------------|--------------|--------|---------|
| 1 | input | VLAN subnet | VLAN gateway | VLAN interface | accept | Allow gateway access |
| 2 | forward | VLAN subnet | WAN gateway | VLAN interface | accept | Allow upstream routing |
| 3 | forward | VLAN subnet | !10.0.0.0/8 | VLAN interface | accept | Allow all non-10.x destinations |
| 4 | forward | VLAN subnet | 10.0.0.0/8 | VLAN interface | drop | Block other lab networks |

The `!` means "NOT" — rule 3 matches traffic going anywhere except 10.x.x.x addresses. This includes both internet destinations and the container network (172.17.0.x).

VLAN 255 (management) gets a different rule 4: **accept** instead of **drop**, allowing it to reach all internal networks.

---

## Lab 10.4 — Create Rules for VLAN 20

### Rule 1: Allow Access to Gateway

1. Navigate to **IP** → **Firewall**

2. Click **Add New** and configure on the **General** tab:
   - **Chain:** input
   - **Src. Address:** 10.10.20.0/24
   - **Dst. Address:** 10.10.20.1
   - **In. Interface:** vlan20

3. Click the **Action** tab:
   - **Action:** accept

4. Add a comment in the **Comment** field: `VLAN 20 gateway access`

5. Click **Apply** and **OK**

### Rule 2: Allow Access to Upstream Router

6. Click **Add New** and configure on the **General** tab:
   - **Chain:** forward
   - **Src. Address:** 10.10.20.0/24
   - **Dst. Address:** [YOUR WAN GATEWAY from Lab 10.1]
   - **In. Interface:** vlan20

7. Click the **Action** tab:
   - **Action:** accept

8. Add a comment: `VLAN 20 upstream access`

9. Click **Apply** and **OK**

### Rule 3: Allow All Non-10.x Destinations

10. Click **Add New** and configure on the **General** tab:
    - **Chain:** forward
    - **Src. Address:** 10.10.20.0/24
    - **Dst. Address:** 10.0.0.0/8
    - **Dst. Address Negation:** ✓ Click the checkbox to add `!`
    - **In. Interface:** vlan20

11. Click the **Action** tab:
    - **Action:** accept

12. Add a comment: `VLAN 20 allow all non-10.x destinations`

13. Click **Apply** and **OK**

### Rule 4: Block Other Lab Networks

14. Click **Add New** and configure on the **General** tab:
    - **Chain:** forward
    - **Src. Address:** 10.10.20.0/24
    - **Dst. Address:** 10.0.0.0/8
    - **Dst. Address Negation:** Leave unchecked (no `!`)
    - **In. Interface:** vlan20

15. Click the **Action** tab:
    - **Action:** drop

16. Add a comment: `VLAN 20 isolate`

17. Click **Apply** and **OK**

18. Select all four VLAN 20 rules using the checkboxes, then drag them as a group so they're positioned after the default rules and the WAN Access rule from Lab 4.

---

## Lab 10.5 — Create Rules for Remaining VLANs

Now that you understand the pattern, create the remaining rules using CLI. Replace `[WAN_GATEWAY]` with your actual gateway IP from Lab 10.1.

### VLAN 30 Rules

```
/ip/firewall/filter/add chain=input src-address=10.10.30.0/24 dst-address=10.10.30.1 in-interface=vlan30 action=accept comment="VLAN 30 gateway access"
/ip/firewall/filter/add chain=forward src-address=10.10.30.0/24 dst-address=[WAN_GATEWAY] in-interface=vlan30 action=accept comment="VLAN 30 upstream access"
/ip/firewall/filter/add chain=forward src-address=10.10.30.0/24 dst-address=!10.0.0.0/8 in-interface=vlan30 action=accept comment="VLAN 30 allow all non-10.x destinations"
/ip/firewall/filter/add chain=forward src-address=10.10.30.0/24 dst-address=10.0.0.0/8 in-interface=vlan30 action=drop comment="VLAN 30 isolate"
```

### VLAN 40 Rules

```
/ip/firewall/filter/add chain=input src-address=10.10.40.0/24 dst-address=10.10.40.1 in-interface=vlan40 action=accept comment="VLAN 40 gateway access"
/ip/firewall/filter/add chain=forward src-address=10.10.40.0/24 dst-address=[WAN_GATEWAY] in-interface=vlan40 action=accept comment="VLAN 40 upstream access"
/ip/firewall/filter/add chain=forward src-address=10.10.40.0/24 dst-address=!10.0.0.0/8 in-interface=vlan40 action=accept comment="VLAN 40 allow all non-10.x destinations"
/ip/firewall/filter/add chain=forward src-address=10.10.40.0/24 dst-address=10.0.0.0/8 in-interface=vlan40 action=drop comment="VLAN 40 isolate"
```

### VLAN 255 Rules (Management — Different!)

VLAN 255 is the management VLAN. It gets access to all internal networks instead of being isolated:

```
/ip/firewall/filter/add chain=input src-address=10.10.255.0/24 dst-address=10.10.255.1 in-interface=vlan255 action=accept comment="VLAN 255 gateway access"
/ip/firewall/filter/add chain=forward src-address=10.10.255.0/24 dst-address=[WAN_GATEWAY] in-interface=vlan255 action=accept comment="VLAN 255 upstream access"
/ip/firewall/filter/add chain=forward src-address=10.10.255.0/24 dst-address=!10.0.0.0/8 in-interface=vlan255 action=accept comment="VLAN 255 allow all non-10.x destinations"
/ip/firewall/filter/add chain=forward src-address=10.10.255.0/24 dst-address=10.0.0.0/8 in-interface=vlan255 action=accept comment="VLAN 255 full access"
```

Note the last rule: **accept** to 10.0.0.0/8 instead of **drop**. This allows VLAN 255 to reach all other lab networks for management purposes.

---

## Lab 10.6 — Verify Rule Order

After creating all rules, verify they're in the correct order. Navigate to **IP** → **Firewall** and review.

The order should flow like this:

1. Default rules (accept established, drop invalid)
2. WAN Access rule (from Lab 4)
3. VLAN 20 rules (4 rules)
4. VLAN 30 rules (4 rules)
5. VLAN 40 rules (4 rules)
6. VLAN 255 rules (4 rules)
7. Default drop rules

If rules are out of order, drag them into position. Rule order matters — traffic matching an earlier rule never reaches later rules.

---

## Lab 10.7 — Understanding the Flow

For each isolated VLAN (20, 30, 40), traffic is processed top-to-bottom:

1. **Gateway access** — Can the device talk to its own gateway? (Yes → accept)
2. **Upstream router** — Can traffic reach the home router? (Yes → accept)
3. **Internet + containers** — Is the destination NOT a 10.x address? (Yes → accept)
4. **Other labs** — Is the destination a 10.x address? (Yes → drop)

For VLAN 255 (management):
- Steps 1-3 are the same
- Step 4: Is the destination a 10.x address? (Yes → **accept** — management can reach everything)

---

## Lab 10.8 — Interface Lists vs Individual Interfaces (Reference)

*Reference: Understanding when to use interface lists in firewall rules.*

Throughout this guide, firewall rules reference individual VLAN interfaces (vlan20, vlan30, etc.) rather than interface lists. This is intentional for learning purposes, but you should understand the tradeoff.

### Individual Interfaces

```
/ip/firewall/filter/add chain=forward src-address=10.10.20.0/24 in-interface=vlan20 action=accept
/ip/firewall/filter/add chain=forward src-address=10.10.30.0/24 in-interface=vlan30 action=accept
```

**Pros:**
- Explicit — you know exactly what each rule affects
- Easier to troubleshoot — "VLAN 30 isn't working" → check vlan30 rules
- Different behavior per VLAN is straightforward

**Cons:**
- More rules as you add VLANs
- Repetitive when all VLANs need the same treatment

### Interface Lists

```
/interface/list/add name=LAB-VLANS
/interface/list/member/add list=LAB-VLANS interface=vlan20
/interface/list/member/add list=LAB-VLANS interface=vlan30
/interface/list/member/add list=LAB-VLANS interface=vlan40

/ip/firewall/filter/add chain=forward in-interface-list=LAB-VLANS action=accept
```

**Pros:**
- Fewer rules — one rule covers all members
- Adding a new VLAN means adding it to the list, not creating new rules
- Scales better for large deployments

**Cons:**
- Less granular — all list members get the same treatment
- Harder to troubleshoot — "which interfaces are in LAB-VLANS again?"
- Exceptions require additional rules

### When to Use Each

| Scenario | Recommendation |
|----------|----------------|
| Learning/lab environment | Individual interfaces |
| Small deployment (<10 VLANs) | Individual interfaces |
| Large deployment (10+ VLANs) | Interface lists |
| All VLANs need identical rules | Interface lists |
| VLANs need different treatment | Individual interfaces |

### Converting Later

If you build with individual interfaces and later want to convert to lists:

1. Create an interface list
2. Add all relevant interfaces to the list
3. Create new rules using the list
4. Test thoroughly
5. Remove the old individual rules

The logic is the same — only the targeting changes.

---

## Lab 10.9 — Firewall Adjustments (Future Reference)

*Reference: Use when deploying to a different network or adding restrictions.*

### If Container Access Stops Working

If you add a more restrictive "drop all else" rule later, you may block container access. Add explicit accept rules before the drop:

```
/ip/firewall/filter/add chain=forward src-address=10.10.20.0/24 dst-address=172.17.0.0/24 action=accept comment="VLAN 20 to containers" place-before=[find comment="VLAN 20 isolate"]
```

The `place-before` parameter ensures the accept rule comes before the drop rule.

### If Your Home Network Uses 10.x Addressing

The "block 10.x" logic assumes your home network doesn't use 10.x addresses. If your home network is 10.0.0.0/8, you'll need to adjust:

1. Change the drop rule to specifically block only your lab subnets
2. Or change your lab addressing to use a different range (172.16.x.x, 192.168.x.x)

### Editing Existing Rules

1. Click on the rule to edit
2. Modify the relevant fields
3. Click **Apply** and **OK**
4. Verify rule order hasn't changed

---

## Lab 10.10 — Port Forwarding (dst-nat)

Port forwarding allows external traffic to reach internal services. This is essential when running your MikroTik as your main router and you need to expose services like game servers, web servers, or remote access.

### How Port Forwarding Works

1. External traffic arrives at your WAN IP on a specific port
2. MikroTik's NAT rule intercepts the traffic
3. Traffic is redirected to an internal IP and port
4. Response traffic is automatically translated back

### Example: Forward Port 8080 to Internal Web Server

**Scenario:** You have a web server at 10.10.20.100 on port 80. You want external users to access it via your public IP on port 8080.

#### Via WinBox

1. Navigate to **IP** → **Firewall** → **NAT**

2. Click **Add New**

3. On the **General** tab:
   - **Chain:** dstnat
   - **Protocol:** tcp
   - **Dst. Port:** 8080
   - **In. Interface:** ether1 (your WAN interface)

4. On the **Action** tab:
   - **Action:** dst-nat
   - **To Addresses:** 10.10.20.100
   - **To Ports:** 80

5. Add a **Comment:** "Port forward 8080 to web server"

6. Click **OK**

#### Via CLI

```
/ip firewall nat add chain=dstnat protocol=tcp dst-port=8080 in-interface=ether1 action=dst-nat to-addresses=10.10.20.100 to-ports=80 comment="Port forward 8080 to web server"
```

### Common Port Forwarding Examples

| Service | External Port | Internal IP | Internal Port | Protocol |
|---------|---------------|-------------|---------------|----------|
| Web server | 8080 | 10.10.20.100 | 80 | tcp |
| HTTPS | 443 | 10.10.20.100 | 443 | tcp |
| SSH | 2222 | 10.10.255.50 | 22 | tcp |
| Minecraft | 25565 | 10.10.20.150 | 25565 | tcp |
| Plex | 32400 | 10.10.20.200 | 32400 | tcp |
| Game server (UDP) | 27015 | 10.10.20.175 | 27015 | udp |

### Forwarding Multiple Ports

For services that need multiple ports (like FTP or some games), create separate rules for each port, or use a port range:

```
/ip firewall nat add chain=dstnat protocol=tcp dst-port=27015-27030 in-interface=ether1 action=dst-nat to-addresses=10.10.20.175 comment="Game server port range"
```

### Same Port, Different External/Internal

You can forward external port 8080 to internal port 80 (as shown above), or keep them the same:

```
# External 80 → Internal 80
/ip firewall nat add chain=dstnat protocol=tcp dst-port=80 in-interface=ether1 action=dst-nat to-addresses=10.10.20.100 to-ports=80

# External 8080 → Internal 80 (different ports)
/ip firewall nat add chain=dstnat protocol=tcp dst-port=8080 in-interface=ether1 action=dst-nat to-addresses=10.10.20.100 to-ports=80
```

### Don't Forget the Firewall

Port forwarding (NAT) redirects traffic, but your firewall filter rules must also allow it. If you have strict input/forward rules, add an accept rule:

```
/ip firewall filter add chain=forward protocol=tcp dst-address=10.10.20.100 dst-port=80 action=accept comment="Allow forwarded web traffic" place-before=[find comment~"drop"]
```

### Verify Port Forwarding

1. Check the NAT rule counters:
   - Navigate to **IP** → **Firewall** → **NAT**
   - Look at the **Bytes** and **Packets** columns
   - If traffic is hitting the rule, counters will increment

2. Test from outside your network:
   - Use a phone on cellular (not Wi-Fi)
   - Try accessing your public IP on the forwarded port
   - Or use an online port checker tool

3. If not working, check:
   - Is the internal host actually listening on that port?
   - Are firewall filter rules blocking the traffic?
   - Is your ISP blocking the port? (common for port 80, 25)
   - Is the in-interface correct?

### Security Considerations

- **Only forward what you need** — every open port is potential attack surface
- **Use non-standard external ports** — forward SSH on 2222 instead of 22 to reduce bot attacks
- **Keep internal services updated** — exposed services should be patched
- **Consider VPN instead** — for personal access, WireGuard (Lab 12-13) is more secure than port forwarding

---

## Lab 10 Summary

At the end of Lab 10, you have:

- ✅ Four VLANs with isolated bridge interfaces
- ✅ DHCP serving unique subnets per VLAN
- ✅ Firewall rules isolating lab VLANs from each other
- ✅ Internet access preserved for all VLANs
- ✅ Container access working from all VLANs
- ✅ Management VLAN (255) with full internal access
- ✅ Backdoor port still functional for emergency access
- ✅ Port forwarding configured for external access to internal services

Your MikroTik is now a fully functional multi-VLAN lab router. Each port provides an isolated network — perfect for testing without impacting other environments.

---

# Lab Notes — Labs 7-10

Print this page or copy to a document for recording important values.

---

**Lab 7 — Interfaces**

| Item | Value |
|------|-------|
| Device Type | ☐ 5-port ☐ 8-port |
| Backdoor Port | |
| Expansion Port | |
| Access Ports Available | |

---

**Lab 8 — DHCP**

| VLAN | Interface | IP Address | Pool Range |
|------|-----------|------------|------------|
| 20 | vlan20 | 10.10.20.1/24 | .10-.250 |
| 30 | vlan30 | 10.10.30.1/24 | .10-.250 |
| 40 | vlan40 | 10.10.40.1/24 | .10-.250 |
| 255 | vlan255 | 10.10.255.1/24 | .10-.250 |

---

**Lab 10 — Firewall**

| Item | Value |
|------|-------|
| Upstream Gateway (quad-zero route) | |
| Management Network (from Lab 4) | |

---

**Testing Results**

| Port | Expected VLAN | IP Received | Container Access |
|------|---------------|-------------|------------------|
| ether2 | 20 | | ☐ Pass ☐ Fail |
| ether3 | 30 | | ☐ Pass ☐ Fail |
| ether4 | 40 (8-port only) | | ☐ Pass ☐ Fail |

---

*Document Version: Draft 3.0*
*Last Updated: May 2026*
