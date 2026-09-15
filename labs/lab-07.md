# Lab 7 — Setting Up Interfaces

*Prerequisites: Lab 5, Lab 6 (backup completed)*

The MikroTik starts with a default configuration (DEFCONF) where ether1 is WAN and all other ports are in a single bridge. This makes them act like a switch, but limits you to one subnet. Time to change that.

> **Warning:** This lab modifies the default bridge configuration. If something goes wrong, you can restore from the backup you created in Lab 6.

---

## Port Availability by Device

Before we begin, understand how port count affects what's available for VLANs:

| Device Type | Models | WAN | Backdoor | Expansion | Available for VLANs |
|-------------|--------|-----|----------|-----------|---------------------|
| 5-port | hAP & hEX series | ether1 | ether4 | ether5 | ether2, ether3 |
| 8-port | L009, RB5009 | ether1 | ether7 | ether8 | ether2-ether6 |

**5-port device constraint:** With only two ports available for VLAN access (ether2, ether3), you can only have two physical access ports. We'll still create all four VLAN bridges and interfaces — VLAN 40 and VLAN 255 will be available via trunk connections even without dedicated physical ports.

> **Important:** Do not reassign your backdoor port (ether4 on 5-port, ether7 on 8-port) to a VLAN. This port stays in the default bridge for emergency access.

---

## Lab 7.1 — Bridge Interfaces

### Explore Current Configuration

1. Navigate to **Bridge** in the left menu.

2. You'll see:
   - **bridge** — The default bridge containing most ports
   - **dockers** — The container bridge we created in Lab 5

3. Click the **Ports** tab at the top.

4. Note which ports are in the default bridge (ether2 through the last port — ether5 on hEX S, ether8 on L009/RB5009, depending on the model — plus the veth interfaces).

### Remove Ports from Default Bridge

We'll remove most ports from the default bridge, keeping only the backdoor port for emergency access.

5. **Important:** Identify your backdoor port (ether4 on hEX S, ether7 on L009/RB5009). We're keeping this in the default bridge.

6. Select **ether2** by clicking the checkbox on the left.

7. Click **Remove** in the toolbar.

8. Repeat for the remaining ports you want to reassign, but **keep your backdoor port in the default bridge**. This ensures you can always access the router at 192.168.88.1.

   For a hEX S, remove ether2 and ether3, keeping ether4 (backdoor) and ether5 (expansion).

   For an L009, remove ether2 through ether6, keeping ether7 (backdoor) and ether8 (expansion).

### Create New Bridges for VLANs

9. Click the **Bridge** tab to return to the bridge list.

10. Click **Add New** and configure:
    - **Name:** `vlan20bridge`
    - **Comment:** `VLAN 20 bridge`
    - **Enabled:** Checked

    > **Note:** Leave the MTU at the default value (1500). Setting the MTU higher than the hardware's L2MTU will cause a red warning banner and can prevent traffic from passing. The L2MTU varies by hardware model (e.g., 1592 on hEX S, higher on L009/RB5009).

11. Click **Apply** and **OK**

12. Create the second bridge using the UI:
    - **Name:** `vlan30bridge`
    - **Comment:** `VLAN 30 bridge`

13. Create the remaining bridges using CLI (faster):

    ```
    /interface/bridge/add name=vlan40bridge comment="VLAN 40 bridge"
    /interface/bridge/add name=vlan255bridge comment="VLAN 255 bridge"
    ```

14. When complete, you should have 6 bridges:
    - bridge (default)
    - dockers (containers)
    - vlan20bridge
    - vlan30bridge
    - vlan40bridge
    - vlan255bridge

### Assign Ports to Bridges

15. Click the **Ports** tab.

16. Click **Add New** and configure:
    - **Interface:** ether2
    - **Bridge:** vlan20bridge
    - **Hardware Offload:** Unchecked
    - **Comment:** `Ether2 VLAN20`

    > **Important:** Disable hardware offload on ports in VLAN-filtered bridges. Hardware offload passes traffic through the switch chip, bypassing the CPU-based VLAN filtering. With it enabled, DHCP and other broadcast traffic may not reach the router's services.

17. Click **Apply** and **OK**

18. Repeat for ether3:
    - **Interface:** ether3
    - **Bridge:** vlan30bridge
    - **Hardware Offload:** Unchecked
    - **Comment:** `Ether3 VLAN30`

19. Add the remaining port assignments using CLI:

    **For 8-port devices (L009, RB5009):**
    ```
    /interface/bridge/port/add bridge=vlan40bridge interface=ether4 hw=no comment="Ether4 VLAN40"
    /interface/bridge/port/add bridge=vlan40bridge interface=ether5 hw=no comment="Ether5 VLAN40"
    /interface/bridge/port/add bridge=vlan40bridge interface=ether6 hw=no comment="Ether6 VLAN40"
    ```

    **For 5-port devices (hAP & hEX series):**
    
    You only have ether2 and ether3 available as access ports. VLAN 40 has no dedicated physical port — it will be accessible via trunk connections configured in Lab 7.4.

    > **Note:** We still create the vlan40bridge and vlan255bridge interfaces. They're used for trunk ports and internal routing even without dedicated physical access ports.

---

## Lab 7.2 — VLAN Interfaces

The next step is creating VLAN interfaces on each bridge. These are the logical interfaces that will receive IP addresses.

### Create VLAN Interfaces

1. Navigate to **Interfaces** in the left menu.

2. You'll see the physical Ethernet interfaces, virtual Ethernet interfaces (veth), and the bridges you created.

3. Click **Add New** and select **VLAN**.

4. Configure:
   - **Name:** vlan20
   - **VLAN ID:** 20
   - **Interface:** vlan20bridge
   - **Comment:** VLAN 20
   - **Enabled:** Checked

5. Click **Apply** and **OK**

6. Create the second VLAN interface using the UI:
   - **Name:** vlan30
   - **VLAN ID:** 30
   - **Interface:** vlan30bridge
   - **Comment:** VLAN 30

7. Create the remaining VLAN interfaces using CLI:

    ```
    /interface/vlan/add name=vlan40 vlan-id=40 interface=vlan40bridge comment="VLAN 40"
    /interface/vlan/add name=vlan255 vlan-id=255 interface=vlan255bridge comment="VLAN 255"
    ```

When complete, you should have four VLAN interfaces:

| Name | VLAN ID | Interface |
|------|---------|-----------|
| vlan20 | 20 | vlan20bridge |
| vlan30 | 30 | vlan30bridge |
| vlan40 | 40 | vlan40bridge |
| vlan255 | 255 | vlan255bridge |

### Add to Interface Lists

The MikroTik uses interface lists for firewall rules. We need to add our new interfaces to the LAN list.

8. Click the **Interface List** tab (second tab from left).

9. Click **Add New** and configure:
   - **Enabled:** Checked
   - **List:** LAN
   - **Interface:** vlan20
   - **Comment:** VLAN 20

10. Click **Apply** and **OK**

11. Repeat for vlan30 using the UI:
    - **List:** LAN
    - **Interface:** vlan30
    - **Comment:** VLAN 30

12. Add the remaining interfaces, bridges, and physical ports using CLI:

    ```
    /interface/list/member/add list=LAN interface=vlan40 comment="VLAN 40"
    /interface/list/member/add list=LAN interface=vlan255 comment="VLAN 255"
    /interface/list/member/add list=LAN interface=vlan20bridge comment="vlan20bridge"
    /interface/list/member/add list=LAN interface=vlan30bridge comment="vlan30bridge"
    /interface/list/member/add list=LAN interface=vlan40bridge comment="vlan40bridge"
    /interface/list/member/add list=LAN interface=vlan255bridge comment="vlan255bridge"
    /interface/list/member/add list=LAN interface=ether2 comment="Ether2 VLAN20"
    /interface/list/member/add list=LAN interface=ether3 comment="Ether3 VLAN30"
    ```

    > **Important:** The physical ports (ether2, ether3, etc.) must be added to the LAN list. The default firewall includes a rule that drops all input traffic not coming from the LAN list. Without adding the physical ports, DHCP requests arriving on those ports will be dropped by the firewall before the DHCP server ever sees them.

    **For 8-port devices (L009, RB5009),** also add:
    ```
    /interface/list/member/add list=LAN interface=ether4 comment="Ether4 VLAN40"
    /interface/list/member/add list=LAN interface=ether5 comment="Ether5 VLAN40"
    /interface/list/member/add list=LAN interface=ether6 comment="Ether6 VLAN40"
    ```

---

## Lab 7.3 — Configuring Access Ports (Switch Ports)

An access port (or switch port) assigns a single VLAN to any device that plugs in — no configuration needed on the device. This is how most enterprise switches work: plug in a computer, sensor, or unmanaged switch, and it gets tagged onto the correct VLAN automatically.

### Understanding the Configuration

To make a port act as an access port, we need:

1. **PVID (Port VLAN ID)** — The VLAN tag assigned to untagged incoming traffic
2. **Frame Types** — Set to only accept untagged frames
3. **VLAN Filtering** — Enabled on the bridge to enforce VLAN membership

### Configure an Access Port

We'll configure ether2 as an access port for VLAN 20.

1. Navigate to **Bridge** → **Ports**

2. Double-click on the **ether2** entry, then click the **VLAN** tab.

3. Configure:
   - **PVID:** 20
   - **Frame Types:** admit-only-untagged-and-priority-tagged

4. Click **Apply** and **OK**

5. Now enable VLAN filtering on the bridge. Navigate to **Bridge** → **Bridge** tab.

6. Double-click on **vlan20bridge** to edit it, then click the **VLAN** tab.

7. Configure:
   - **VLAN Filtering:** Checked

8. Click **Apply** and **OK**

> **Note:** RouterOS automatically creates the VLAN table entries when you set the PVID on the port. You do not need to manually add VLAN entries in the Bridge → VLANs tab.

### CLI Equivalent

For the remaining access ports, use CLI:

```
# Configure ether3 as access port for VLAN 30
/interface/bridge/port/set [find interface=ether3] pvid=30 frame-types=admit-only-untagged-and-priority-tagged
/interface/bridge/set vlan30bridge vlan-filtering=yes

# Configure ether4 as access port for VLAN 40
/interface/bridge/port/set [find interface=ether4] pvid=40 frame-types=admit-only-untagged-and-priority-tagged
/interface/bridge/set vlan40bridge vlan-filtering=yes
```

### What This Accomplishes

Any device plugged into ether2 will:
- Have its traffic tagged with VLAN 20 on ingress
- Receive untagged traffic destined for VLAN 20 on egress
- Work without any VLAN configuration on the device itself

This is identical to how you'd configure an access port on a Cisco, Juniper, or Ruckus ICX switch.

---

## Lab 7.4 — Configuring Trunk Ports

A trunk port carries multiple VLANs to another switch using 802.1Q tags. The remote switch (MikroTik, Cisco, Ruckus ICX, etc.) handles tagging on its own ports.

### Understanding Trunk Configuration

Unlike access ports where we assign one VLAN untagged, trunk ports:
- Carry multiple VLANs as tagged traffic
- May also carry one VLAN untagged (often for management)
- Require VLAN interfaces created on the physical port

### Scenario

We'll configure the expansion port (ether8 on L009, ether5 on hEX S) as a trunk carrying:
- VLAN 20, 30, 40 as tagged traffic
- VLAN 255 as untagged (management)

### Remove the Expansion Port from the Default Bridge

1. Navigate to **Bridge** → **Ports**.

2. If your expansion port (ether5 on hEX S, ether8 on L009) is still in the default bridge, select it and click **Remove**.

   > **Why:** A port cannot belong to two bridges simultaneously. If ether5/ether8 is still in the default bridge from the factory configuration, you must remove it before assigning it to vlan255bridge.

### Create VLAN Interfaces on the Trunk Port

3. Open a **Terminal** and create VLAN interfaces directly on the physical port:

   ```
   /interface/vlan/add interface=ether8 vlan-id=20 name=ether8-vlan20 comment="Trunk VLAN 20"
   /interface/vlan/add interface=ether8 vlan-id=30 name=ether8-vlan30 comment="Trunk VLAN 30"
   /interface/vlan/add interface=ether8 vlan-id=40 name=ether8-vlan40 comment="Trunk VLAN 40"
   ```

   > **Note:** Adjust `ether8` to match your expansion port. For a hEX S, replace `ether8` with `ether5` in all commands.

### Add VLAN Interfaces to Bridges

4. Connect each trunk VLAN interface to its corresponding bridge:

   ```
   /interface/bridge/port/add bridge=vlan20bridge interface=ether8-vlan20 comment="Trunk to VLAN 20"
   /interface/bridge/port/add bridge=vlan30bridge interface=ether8-vlan30 comment="Trunk to VLAN 30"
   /interface/bridge/port/add bridge=vlan40bridge interface=ether8-vlan40 comment="Trunk to VLAN 40"
   ```

### Configure Untagged Management VLAN

5. Add the physical port to vlan255bridge for untagged management traffic:

   ```
   /interface/bridge/port/add bridge=vlan255bridge interface=ether8 pvid=255 frame-types=admit-only-untagged-and-priority-tagged comment="Trunk untagged VLAN 255"
   ```

   > **Note:** For hEX S, replace `ether8` with `ether5`.

6. Enable VLAN filtering on vlan255bridge and set the bridge PVID:

   ```
   /interface/bridge/set vlan255bridge vlan-filtering=yes pvid=255
   ```

7. Add the VLAN table entry so vlan255bridge knows ether8 is an untagged member of VLAN 255:

   ```
   /interface/bridge/vlan/add bridge=vlan255bridge vlan-ids=255 untagged=ether8
   ```

   > **Critical:** Without steps 6 and 7, untagged frames arriving on ether8 will never reach the vlan255 interface or DHCP server — VLAN filtering must be enabled and the VLAN table entry must exist for the bridge to process untagged traffic correctly. These three steps work together: the bridge port PVID tags incoming untagged frames as VLAN 255, VLAN filtering enables the VLAN table, and the VLAN table entry confirms ether8 is a legitimate untagged member of VLAN 255.

### Why frame-types Matters

This setting is critical when mixing tagged and untagged traffic on the same physical port:

| Setting | Behavior |
|---------|----------|
| admit-all | Accept both tagged and untagged frames (can cause issues) |
| admit-only-untagged-and-priority-tagged | Only accept untagged frames — tagged frames pass to VLAN interfaces |
| admit-only-vlan-tagged | Only accept tagged frames |

By setting `admit-only-untagged-and-priority-tagged` on the physical port's bridge membership, we ensure:
- **Untagged traffic** lands on vlan255bridge (management)
- **Tagged traffic** passes through to the VLAN interfaces (ether8-vlan20, etc.)

### Next Steps

The MikroTik side of the trunk is now configured. To configure the remote switch (ICX, Cisco, or other enterprise switch), proceed to **Lab 18 — Enterprise Switch Integration**.

---
