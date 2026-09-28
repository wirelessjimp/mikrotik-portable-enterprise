# Lab 19 — Enterprise Switch Integration

*Prerequisites: Lab 7 (trunk ports configured), Lab 8 (DHCP configured)*

This lab connects an enterprise switch to your MikroTik via trunk, extending VLANs to additional ports. This is the infrastructure needed to connect enterprise APs that require PoE and VLAN tagging.

> **Tested Configuration:** This lab was developed using a RUCKUS ICX series switch. The concepts apply to any managed switch — only the CLI syntax differs. Refer to your vendor's documentation for specific commands.

---

## Lab 19.1 — Understanding Trunk Ports

Before connecting the switch, let's clarify what we're building.

### VLAN Tagging Concepts

| Term | Meaning | Also Called |
|------|---------|-------------|
| Tagged | VLAN ID is included in the Ethernet frame | 802.1Q, trunk |
| Untagged | No VLAN ID in the frame; switch assigns the port's native VLAN | Access, native |
| Native VLAN | The VLAN used for untagged traffic on a trunk | PVID, default VLAN |

### What a Trunk Port Does

A trunk port carries multiple VLANs over a single cable:
- **Tagged VLANs:** Frames include the VLAN ID; both ends must agree on tagging
- **Native/Untagged VLAN:** Frames have no tag; used for management traffic

### Our Trunk Design

| VLAN | Purpose | Tagged/Untagged |
|------|---------|-----------------|
| 20 | Data | Tagged |
| 30 | Voice | Tagged |
| 40 | IoT | Tagged |
| 255 | Management | Untagged (native) |

The switch receives an IP address on VLAN 255 (untagged), and passes VLANs 20, 30, 40 to access ports or downstream devices (like an AP).

---

## Lab 19.2 — MikroTik Trunk Port Verification

If you completed Lab 7.4, your expansion port is already configured as a trunk. Let's verify.

### Check VLAN Interfaces

1. On your main router, navigate to **Interfaces**

2. Verify you have VLAN interfaces on your expansion port:

   For 5-port devices (hEX, hAP), look for:
   - ether5-vlan20 (or similar naming)
   - ether5-vlan30
   - ether5-vlan40

   For 8-port devices (L009, RB5009), look for:
   - ether8-vlan20
   - ether8-vlan30
   - ether8-vlan40

3. If these don't exist, create them via **Terminal**:

   ```
   # For 8-port devices (adjust interface name for 5-port)
   /interface/vlan/add interface=ether8 vlan-id=20 name=ether8-vlan20 comment="Trunk VLAN 20"
   /interface/vlan/add interface=ether8 vlan-id=30 name=ether8-vlan30 comment="Trunk VLAN 30"
   /interface/vlan/add interface=ether8 vlan-id=40 name=ether8-vlan40 comment="Trunk VLAN 40"
   ```

### Verify Bridge Port Assignment

4. Navigate to **Bridge** → **Ports**

5. Verify each trunk VLAN interface is added to its corresponding bridge:
   - ether8-vlan20 → vlan20bridge
   - ether8-vlan30 → vlan30bridge
   - ether8-vlan40 → vlan40bridge

6. Verify the physical port (ether8) is in vlan255bridge for native/untagged traffic.

### What This Looks Like

When properly configured, traffic flows like this:

```
Switch                        MikroTik
──────                        ────────
VLAN 20 tagged    ────────►   ether8 → VLAN interface → vlan20bridge
VLAN 30 tagged    ────────►   ether8 → VLAN interface → vlan30bridge  
VLAN 40 tagged    ────────►   ether8 → VLAN interface → vlan40bridge
VLAN 255 untagged ────────►   ether8 → directly to vlan255bridge
```

---

## Lab 19.3 — Switch Configuration Concepts

Every managed switch needs the same basic configuration for this lab. The CLI syntax varies by vendor, but the concepts are universal.

### What You Need to Configure

**1. Create VLANs**

Create VLANs 20, 30, 40, and 255 on the switch. Some switches create VLAN 1 by default — we won't use it.

**2. Configure the Uplink Port (to MikroTik)**

The uplink port must:
- Tag VLANs 20, 30, 40 (these frames get an 802.1Q header)
- Leave VLAN 255 untagged/native (these frames have no header)

This is sometimes called:
- "Trunk port" with native VLAN (Cisco)
- "Tagged/untagged" membership (HP/Aruba, Ruckus)
- "Dual-mode" (some vendors)

**3. Configure Access Ports (for testing)**

Access ports connect end devices:
- Assign each port to a single VLAN
- Traffic leaves the port untagged
- Traffic entering the port gets assigned to that VLAN

**4. Configure AP Ports (if connecting an AP)**

AP ports typically need:
- Untagged VLAN for AP management (so the AP gets an IP)
- Tagged VLANs for wireless clients (each SSID can use a different VLAN)

**5. Configure Switch Management IP**

The switch needs an IP address on VLAN 255 (the management VLAN) so you can manage it. Options:
- DHCP: Switch requests an address from MikroTik's DHCP server
- Static: Manually assign an IP in the 10.10.255.x range

### Configuration Reference Table

Use this table when configuring your switch. Translate to your vendor's syntax.

| Port | VLANs Tagged | VLAN Untagged | Purpose |
|------|--------------|---------------|---------|
| Uplink (to MikroTik) | 20, 30, 40 | 255 | Trunk to router |
| Access port example | — | 20 | End device on VLAN 20 |
| Access port example | — | 255 | End device on management |
| AP port | 20, 30 | 255 | Enterprise AP |

---

## Lab 19.4 — Physical Connection

1. Power on your switch.

2. Connect your laptop to the switch (any port) for initial configuration.

3. Access the switch via:
   - **Console cable:** Serial connection to the console port
   - **Default IP:** Many switches have a default IP (check documentation)
   - **DHCP:** Some switches request DHCP; check MikroTik's DHCP leases

4. Log in with default credentials (check your switch's documentation).

5. Perform basic setup:
   - Set hostname/identity
   - Set admin password
   - Configure management IP (if not using DHCP)

6. Save the configuration.

---

## Lab 19.5 — Connect Switch to MikroTik

### Physical Connection

1. Connect the switch's uplink port to your MikroTik's expansion port (ether5 or ether8).

2. The switch should now:
   - Receive a DHCP lease on VLAN 255 (if configured for DHCP)
   - Be reachable at its management IP

### Verify from MikroTik

3. On your MikroTik, navigate to **IP** → **DHCP Server** → **Leases**

4. Look for a new lease on the vlan255bridge DHCP server — this is your switch.

5. Record the switch's IP address:

   > **Switch IP:** ________________________________

### Verify from Switch

6. From the switch CLI, verify you can ping the MikroTik gateway:
   
   ```
   ping 10.10.255.1
   ```

7. Verify you can ping an external address (confirms routing works):
   
   ```
   ping 8.8.8.8
   ```

---

## Lab 19.6 — Test VLAN Connectivity

### Test Access Port on VLAN 20

1. Configure a switch port as an access port on VLAN 20 (untagged).

2. Connect your laptop to that port.

3. Your laptop should receive an IP in **10.10.20.0/24** from MikroTik's VLAN 20 DHCP server.

4. Verify:
   - Ping 10.10.20.1 (MikroTik gateway for VLAN 20) ✓
   - Ping 8.8.8.8 (internet) ✓

### Test Access Port on VLAN 255

5. Move your laptop to a switch port configured for VLAN 255 (untagged).

6. Your laptop should receive an IP in **10.10.255.0/24**.

7. Verify:
   - Ping 10.10.255.1 (MikroTik gateway) ✓
   - Ping the switch at its management IP ✓

### Test Cross-VLAN Isolation

8. With your laptop on VLAN 20, try to ping a device on VLAN 255.

9. This should **fail** (blocked by firewall rules from Lab 10).

10. Cross-VLAN communication should only work where explicitly allowed.

---

## Lab 19.7 — Configure AP Port

If you're connecting an enterprise AP in the next lab, configure a port for it now.

### AP Port Requirements

Most enterprise APs need:
- **Untagged VLAN** for management: The AP gets its IP here
- **Tagged VLANs** for wireless clients: Different SSIDs map to different VLANs

### Configure the Port

1. Choose a switch port for the AP.

2. Configure it with:
   - VLAN 255 untagged (native) — AP management
   - VLANs 20, 30 tagged — for wireless client VLANs

3. If the switch supports PoE:
   - Enable PoE on this port
   - Verify power budget is sufficient for your AP

### Record the Configuration

| Setting | Value |
|---------|-------|
| AP Port Number | |
| Untagged VLAN | 255 |
| Tagged VLANs | 20, 30 |
| PoE Enabled | ☐ Yes ☐ No |

---

## Lab 19.8 — MikroTik Switch (SwOS) — Optional

If you have a MikroTik CSS switch (runs SwOS, not RouterOS), the configuration method is different. SwOS uses a web interface.

### Access SwOS

1. Connect the switch to power and to your MikroTik's expansion port.

2. The switch will request DHCP. Check **IP** → **DHCP Server** → **Leases** on your MikroTik.

3. Access the switch web interface at its assigned IP.

4. Default credentials: admin / (blank)

### Configure VLANs in SwOS

1. Click the **VLAN** tab.

2. For the uplink port (typically Port 1):
   - **VLAN Mode:** Enabled
   - **VLAN Receive:** any
   - **Default VLAN ID:** 255
   - **Force VLAN ID:** Checked
   - **VLAN Header:** add if missing

3. For access ports:
   - **VLAN Mode:** Enabled
   - **VLAN Receive:** only untagged
   - **Default VLAN ID:** [Desired VLAN]
   - **Force VLAN ID:** Unchecked
   - **VLAN Header:** leave as is

4. Click **Apply All**

### Add VLANs to SwOS

5. Click the **VLANs** tab (plural).

6. Click **Append** to add each VLAN: 20, 30, 40, 255

7. For each VLAN, configure port membership:
   - Uplink port: Tagged (except 255 which is untagged)
   - Access ports: Their assigned VLAN as untagged, empty for others

8. Click **Apply All**

9. Click **System** and verify **Independent VLAN Lookup** is enabled.

10. Click **Apply All**

---

## Lab 19 Summary

You now have:

- ✅ Trunk connection between MikroTik and enterprise switch
- ✅ VLANs 20, 30, 40 tagged; VLAN 255 native/untagged
- ✅ Switch receiving management IP on VLAN 255
- ✅ Access ports tested for VLAN assignment
- ✅ AP port prepared (if applicable)

Your network can now support enterprise APs and other devices that require PoE and VLAN tagging.

---

## Lab Notes — Lab 19

| Item | Value |
|------|-------|
| Switch Make/Model | |
| Switch Management IP | |
| Switch Uplink Port | |
| AP Port Number | |

**VLAN Verification:**

| Test | Result |
|------|--------|
| Switch gets IP on VLAN 255 | ☐ Pass ☐ Fail |
| Laptop gets IP on VLAN 20 access port | ☐ Pass ☐ Fail |
| Laptop gets IP on VLAN 255 access port | ☐ Pass ☐ Fail |
| Cross-VLAN traffic blocked | ☐ Pass ☐ Fail |

---
