# Lab 16 — Second MikroTik Device (mAP)

*Prerequisites: Lab 7 (trunk ports configured), Lab 12 (WireGuard server), Lab 6 (backup completed)*

This lab builds out a second MikroTik device as a portable, fully-featured extension of your network. When complete, you'll have a device you can take anywhere — plug it into any network with internet access and it tunnels home, giving you secure access to your lab.

We're using the mAP 2nd for this lab. It's small, cheap, runs on USB power, and has Wi-Fi built in. The same process works for any RouterOS device — if you want more power, the hAP ac² hAP ax² are solid upgrades with better Wi-Fi and more RAM, but a little larger.

### What We're Building

| Feature | Purpose |
|---------|---------|
| Fallback config | Emergency access when not connected to main router |
| Trunk connection | Carry all VLANs back to main router |
| WireGuard tunnel | Secure connection home from anywhere |
| RoMON | Manage both devices from one connection |
| Management Wi-Fi | Wireless access to the network on VLAN 255 |

---

## Lab 16.1 — Initial mAP Access via WinBox

### Physical Connection

1. Connect the mAP's ETH1 port to your main router's expansion port (ether5 on 5-port devices, ether8 on 8-port devices).

   > **Power:** The mAP can be powered via PoE from the router (if your router supports PoE-out - both the L009, hEX S, and RB5009 do), USB, or the included adapter. For this lab, any power source works.

2. Wait for the mAP to boot — the PWR light will go solid green.

### Connect via WinBox

3. Move your laptop's ethernet cable from the main router to **ether2** on the mAP. This gives you direct Layer 2 access to the mAP so WinBox can discover it.

4. Open WinBox on your computer.

5. Click the **Neighbors** tab.

6. You should see the mAP listed (it will show its MAC address and possibly an IP in 192.168.88.x).

7. Click on the mAP's **MAC address** to select it.

   > **Important:** Select the MAC address, not the IP address. At this stage the mAP may not have an IP on your subnet, and connecting by MAC ensures WinBox can reach it regardless.

8. Enter credentials:
   - **Login:** admin
   - **Password:** (leave blank for factory default)

9. Click **Connect**

### Initial Configuration

10. If prompted about the default configuration, click **OK** to keep it for now (we'll reset it shortly).

11. RouterOS will prompt you to change the default password. Set a new password.

    > **Tip:** Use the same password as your main router so you're not juggling multiple credentials during the lab.

12. Navigate to **System** → **Identity**

13. Set the identity to something memorable: `mAP-Remote`

14. Click **Apply** and then **OK**

### Update RouterOS

15. Navigate to **System** → **Packages**

16. Click **Check For Updates**

17. Ensure **Channel** is set to **stable**

18. If updates are available, click **Download & Install**

19. Wait for the mAP to reboot (about 60 seconds).

20. WinBox will attempt to reconnect automatically — close that window. Open a fresh WinBox session, click the **Neighbors** tab, select the mAP by MAC address, and log in with your new password.

---

## Lab 16.2 — Reset to Blank Configuration

The mAP ships with a default configuration designed for "plug and play home router" use — NAT, DHCP server, firewall rules, bridged ports. That's the opposite of what we need (a trunk endpoint with no local NAT or DHCP).

We'll wipe everything and build exactly what we need from scratch.

> **Note:** This will temporarily disconnect your WinBox session. After the reset, reconnect by MAC address — WinBox can discover and connect to MikroTik devices by MAC address even with a completely blank configuration.

### Perform the Reset

1. In WinBox, navigate to **System** → **Reset Configuration**

2. Configure:
   - **No Default Configuration:** ✓ Checked
   - **Do Not Backup:** ✓ Checked

3. Click **Reset Configuration**

4. Click **OK** to confirm.

5. The mAP will reboot with a completely blank configuration. Close the WinBox session — the credentials you just set no longer exist on the device.

### Reconnect After Reset

6. Open WinBox, click the **Neighbors** tab, and select the mAP by MAC address — the same way you connected in Lab 16.1.

7. Connect with:
   - **Login:** admin
   - **Password:** (blank)

8. You'll be prompted to set a password. Set one and record it in your lab notes.

> **Why MAC address still works:** Even with a blank configuration, WinBox can discover and connect to MikroTik devices by MAC address over Layer 2. No IP address or DHCP required.

---

## Lab 16.3 — Build Fallback Configuration

Before we configure the trunk connection, we'll build a minimal "standalone fallback" configuration. This ensures you can always access the mAP even when it's not connected to your main router — useful for troubleshooting or using the mAP as a standalone device while traveling.

We'll use 192.168.89.0/27 for this fallback network — similar to the default 192.168.88.x but clearly distinguishable.

### Create Fallback Bridge

1. Navigate to **Bridge**

2. Click **New**

3. Configure:
   - **Name:** br-fallback
   - **Comment:** Standalone fallback bridge

4. Click **Apply** and then **OK**

### Add Ports to Fallback Bridge

5. From the Bridge window, click the **Ports** tab

6. Click **New**:
   - **Interface:** ether2
   - **Bridge:** br-fallback
   - **Comment:** Fallback ETH2

7. Click **Apply** and then **OK**

   > **Note:** Adding ether2 to the bridge will cause your WinBox session to disconnect. Auto-reconnect will not work here — close the WinBox window and open a fresh session, connecting by MAC address with your updated password.

### Configure Fallback IP Address

8. Navigate to **IP** → **Addresses**

9. Click **New**:
   - **Address:** 192.168.89.1/27
   - **Interface:** br-fallback
   - **Comment:** Fallback management IP

10. Click **Apply** and then **OK**

### Create Fallback DHCP Pool

11. Navigate to **IP** → **Pool**

12. Click **New**:
    - **Name:** fallback-pool
    - **Addresses:** 192.168.89.10-192.168.89.30
    - **Comment:** Fallback DHCP pool

13. Click **Apply** and then **OK**

### Create Fallback DHCP Server

14. Navigate to **IP** → **DHCP Server**

15. Click **New**:
    - **Name:** fallback-dhcp
    - **Interface:** br-fallback
    - **Address Pool:** fallback-pool
    - **Lease Time:** 01:00:00
    - **Add ARP For Leases:** ✓ Checked

16. Click **Apply** and then **OK**

17. In the **DHCP Server** window, click the **Networks** tab

18. Click **New**:
    - **Address:** 192.168.89.0/27
    - **Gateway:** 192.168.89.1
    - **DNS Servers:** 192.168.89.1
    - **Comment:** Fallback network

19. Click **Apply** and then **OK**

### Configure Fallback Wi-Fi

20. Navigate to **Wireless** → **Wireless** → **Security Profiles**

21. Click **New**:
    - **Name:** fallback-psk
    - **Mode:** dynamic keys
    - **Authentication Types:** WPA2 PSK
    - **WPA2 Pre-Shared Key:** [Create a password and record it in your lab notes]

22. Click **Apply** and then **OK**

23. Navigate to **Wireless** → **Wireless** (the submenu, not the top-level menu)

    > **Note:** On RouterOS 7.x you'll see both a **WiFi** menu and a **Wireless** menu. Use **Wireless** → **Wireless** → **WiFi Interfaces** to access the legacy wireless interface where wlan1 lives.

24. Double-click **wlan1** to edit it and click the **Wireless** tab:
    - **Mode:** ap bridge
    - **Band:** 2GHz-only-N
    - **Channel Width:** 20MHz
    - **Frequency:** 2412 (or **auto** if in a class)
    - **SSID:** mAP-Fallback
    - **Security Profile:** fallback-psk
    - **Country:** united states (or your country)

25. Click **Apply** and then **OK**

26. On the **WiFi Interfaces** interface list, select **wlan1** and click **Enable**.

### Add Wi-Fi to Fallback Bridge

27. Navigate to **Bridge** → **Ports**

28. Click **New**:
    - **Interface:** wlan1
    - **Bridge:** br-fallback
    - **Comment:** Fallback Wi-Fi

29. Click **Apply** and then **OK**

### Test Fallback Configuration

30. Set your laptop back to DHCP (if it isn't already).

31. Disconnect and reconnect the ethernet cable to the mAP's ether2 port.

32. Your laptop should receive an IP in the **192.168.89.x** range.

33. WinBox should reconnect automatically to **192.168.89.1**. If it doesn't, open a fresh session and connect by IP or MAC address.

34. You can also test the Wi-Fi:
    - Connect to **mAP-Fallback** SSID using the password you set
    - You should get a 192.168.89.x address
    - You should be able to reach 192.168.89.1

> **What you've built:** A standalone access point with its own DHCP server. This works even when the mAP isn't connected to your main router. Think of it as "emergency mode."
>
> **What's missing:** Internet access. The mAP can hand out addresses and you can reach its management interface, but there's no upstream path to the internet yet. That's what Lab 16.4 builds — the trunk connection back to your main router that carries all your VLANs and provides the gateway.

---

## Lab 16.4 — Configure WAN and Trunk Connection

Now we configure the mAP to get internet from whatever network it's plugged into, and carry tagged VLANs back to your main router when connected via trunk.

> **All steps in Lab 16.4 are performed on the mAP.** Your main router is already configured from Labs 7-10. Make sure your WinBox session is connected to the mAP (192.168.89.1 or by MAC address), not your main router.

### The Design

The mAP uses ether1 for everything upstream:
- When plugged into your main router's trunk port (ether8/5), it gets a management IP from the VLAN 255 DHCP server and carries tagged VLANs 20/30/40
- When plugged into any other network (hotel, coffee shop, client site), it gets a WAN IP via DHCP from that network
- A WireGuard tunnel (Lab 16.5) handles secure connectivity back to your lab in travel mode

### Create VLAN Interfaces

1. Navigate to **Interfaces**

2. Click **New** → **VLAN**

3. Configure:
   - **Name:** vlan20
   - **VLAN ID:** 20
   - **Interface:** ether1
   - **Comment:** VLAN 20 from trunk

4. Click **Apply** and then **OK**

5. Create the remaining VLANs — open **Terminal** and run:

   ```
   /interface/vlan/add name=vlan30 vlan-id=30 interface=ether1 comment="VLAN 30 from trunk"
   /interface/vlan/add name=vlan40 vlan-id=40 interface=ether1 comment="VLAN 40 from trunk"
   ```

   > **Note:** Do not create a vlan255 sub-interface. VLAN 255 is the native/untagged VLAN on this trunk — ether1 carries it directly without tagging.

### Create Management Bridge

6. Navigate to **Bridge** → **Bridge**

7. Click **New**:
   - **Name:** br-mgmt
   - **Comment:** Management bridge VLAN 255

8. Click **Apply** and then **OK**

9. Click the **Ports** tab

10. Click **New**:
    - **Interface:** ether1
    - **Bridge:** br-mgmt
    - **Comment:** Trunk native VLAN 255

11. Click **Apply** and then **OK**

### Configure WAN DHCP Client

12. Navigate to **IP** → **DHCP Client**

13. Click **New**:
    - **Interface:** br-mgmt
    - **Use Peer DNS:** yes
    - **Use Peer NTP:** yes
    - **Add Default Route:** yes
    - **Comment:** WAN DHCP client

14. Click **Apply** and then **OK**

    > **How this works:** When plugged into your main router's trunk port, br-mgmt receives untagged frames on VLAN 255 and the DHCP client picks up an address in the 10.10.255.x range. When plugged into any other network, it picks up whatever address that network hands out. The default route updates automatically in both cases.

### Configure DNS

> **⚑ Flag:** If internet access isn't working after completing Lab 16.4, come back and verify this step first — DNS is a common culprit.

15. Navigate to **IP** → **DNS**

16. Set **Servers:** 10.10.255.1

17. Click **Apply** and then **OK**

### Test Trunk Connectivity

18. Plug the mAP's ether1 into your main router's expansion port (ether8/5).

19. Navigate to **IP** → **DHCP Client** and verify the status shows **bound** with an address in the **10.10.255.x** range.

20. Navigate to **Tools** → **Ping**

21. Ping **10.10.255.1** (your main router) — this should succeed.

22. Ping **8.8.8.8** to verify internet access through the main router.

> **If pings fail:** Check that ether5 on your main router has VLAN 255 configured as the native/untagged VLAN with PVID 255 and VLAN filtering enabled on the bridge (Lab 7.4).

---

## Lab 16.5 — Configure WireGuard Tunnel

Now we configure the mAP to tunnel back to your main router via WireGuard. This is what makes the mAP a travel router — plug it into any network with internet access and it tunnels home automatically.

### Prerequisites

Before starting, verify:
- The mAP has internet access (Lab 16.4 complete, ping 8.8.8.8 works)
- Your main router's WireGuard server is running on port 51820 (Lab 12)
- **UDP port 51820 is port-forwarded** on your upstream router (the router that provides internet to your main router) to your main router's WAN IP. Without this, the tunnel cannot form when the mAP is on an external network.

### Two WinBox Sessions

Lab 16.5 requires simultaneous access to both the mAP and the main router. Before starting:

1. Move your laptop cable from the mAP to your **main router's backdoor port** (ether8 on the L009, ether4 on the hEX S, 192.168.88.1)
2. Open a WinBox session to your **main router** via Neighbors or IP
3. Open a **second WinBox session** to the mAP by typing **10.10.255.x** (the IP the mAP received from the main router's DHCP) directly into the Connect To field — it won't appear in Neighbors from this connection
   > If you can't find the IP address for your mAP, on your main router click on **IP** → **DHCP Server** → **Leases** and the mAP will be listed there. This is the IP address to connect to.
5. Log in with your mAP password

Keep both sessions open throughout this lab.

### Create WireGuard Interface

> **You are on the mAP** for steps 1-8.

1. Navigate to **WireGuard**

2. Click **New**:
   - **Name:** wg-home
   - **Listen Port:** 51820
   - **MTU:** 1420

3. Click **Apply** and then **OK**

4. Double-click on **wg-home** to view it

5. Copy the **Public Key** field and record it in your lab notes — you'll need it in step 13.

   > **mAP WireGuard Public Key:** ________________________________

### Configure WireGuard IP

6. Navigate to **IP** → **Addresses**

7. Click **New**:
   - **Address:** 10.255.255.2/24
   - **Interface:** wg-home
   - **Comment:** WireGuard tunnel IP

8. Click **Apply** and then **OK**

### Add Peer on mAP (pointing to main router)

9. Navigate to **WireGuard** → **Peers** tab

10. Click **New**:
    - **Interface:** wg-home
    - **Public Key:** 
       - Switch to your **main router** WinBox session → navigate to **WireGuard** → double-click **wg-server** → copy the **Public Key** field. This is the **interface** public key, not a peer key.
       - Switch back to your mAP and paste the copied key into the **Public Key** window.
    - **Endpoint:** 
       - Switch to your **main router** WinBox session → navigate to **IP** → **Cloud** → copy the **DNS Name** field.
       - Switch back to your mAP and paste it, adding `:51820` at the end.
    - **Allowed Address:** 10.255.255.0/24, 10.10.0.0/16
    - **Persistent Keepalive:** 00:00:25
    - **Comment:** Main router

11. Click **Apply** and then **OK**

### Add Peer on Main Router (pointing to mAP)

> **Switch to your main router WinBox session** for steps 12-14.

12. Navigate to **WireGuard** → **Peers**

13. Click **New**:
    - **Interface:** wg-server
    - **Public Key:** [mAP's public key from step 5]
    - **Allowed Address:** 10.255.255.2/32
    - **Comment:** mAP remote

14. Click **Apply** and then **OK**

### Test WireGuard Tunnel

> **Back on the mAP** for the remaining steps.

15. Navigate to **Tools** → **Ping**

16. Ping **10.255.255.1** (main router's WireGuard IP)

17. Navigate to **WireGuard** → **Peers**

18. Check the **Last Handshake** column — it should show a recent timestamp (within the last minute) and non-zero Rx/Tx counters.

> **If no handshake:**
> - Verify the public keys match on both sides — the mAP peer must have the **wg-server interface** public key, not a peer public key
> - Verify UDP 51820 is port-forwarded on your upstream router to your main router's WAN IP
> - Check that your main router's firewall allows UDP 51820 inbound (Lab 12.3)
> - If the mAP is on the same network as the main router, NAT hairpinning may prevent the tunnel from forming — test with the mAP on a separate internet connection

---

## Lab 16.6 — Configure RoMON

RoMON (Router Management Overlay Network) lets you manage multiple MikroTik devices through a single connection. Once configured, you can access any RoMON-enabled device on the network without needing direct IP connectivity to it.

### What RoMON Does

Think of RoMON as a Layer 2 management tunnel between MikroTik devices. When you connect to any device in the RoMON network:
- You can "discover" all other RoMON-enabled devices
- You can open a management session to any discovered device
- This works even if you don't have IP routing to that device

**Use case:** You're connected to the mAP via the fallback Wi-Fi. Through RoMON, you can manage the main router without knowing its IP address or having a route to it — as long as there's a Layer 2 path (the trunk) between them.

### Enable RoMON on mAP

1. Navigate to **Tools** → **RoMON**

2. Click the **Settings** button (or just click in the main RoMON window)

3. Configure:
   - **Enabled:** ✓ Checked
   - **Secrets:** Enter the same RoMON secret you created on the main router in Lab 3. Check your lab notes if you don't remember it.
   - **ID:** Leave as default (MAC-based)

4. Click **Apply**

### Configure RoMON Ports

5. Click the **Ports** tab and verify the default entry shows **Interface: all** — this means RoMON will operate on every interface. No changes needed.

### Enable RoMON on Main Router

6. Connect to your main router via WinBox

7. Navigate to **Tools** → **RoMON**

12. Click **Settings**:
    - Verify the following:
       - **Enabled:** ✓ Checked
       - **Secrets:** [Populated]

16. Click the **Ports** tab and verify the default entry shows **Interface: all** — this means RoMON will operate on every interface. No changes needed.

17. Click **Apply** and then **OK** if any changes were made.

### Discover Devices via RoMON

17. On either device in WinBox, click the **Discover** tab (or button)

19. You should see both devices listed with their MAC addresses and identities.

20. To connect to a device: select it and click **Connect** — a new WinBox session opens to that device.

### Using RoMON from WinBox Neighbors

Even simpler — WinBox's Neighbors tab can use RoMON:

1. Open WinBox

2. Connect to either device normally

3. Click **Neighbors** in the left menu

4. Devices reachable via RoMON appear here

5. Double-click any device to open a new WinBox window to it

This means: connect to the mAP via fallback Wi-Fi, then manage your main router through RoMON without any additional network configuration.

---

## Lab 16.7 — Configure Management Wi-Fi on VLAN 255

The fallback Wi-Fi (mAP-Fallback) is for emergency standalone access. Now we'll add a second SSID that puts clients on the management VLAN (255), giving them access to the full network when the mAP is connected to the main router.

### Create Management Wi-Fi Security Profile

1. Navigate to **Wireless** → **Wireless** → **Security Profiles**

2. Click **New**:
   - **Name:** mgmt-security
   - **Mode:** dynamic keys
   - **Authentication Types:** WPA2 PSK
   - **WPA2 Pre-Shared Key:** [Create a strong password — different from fallback]

3. Click **Apply** and then **OK**

### Create Virtual AP for Management

4. Navigate to **Wireless** → **WiFi Interfaces**

5. Click **New** → **Virtual**

6. Configure:
   - **Name:** wlan2
   - **Mode:** ap bridge
   - **Master Interface:** wlan1
   - **SSID:** Student[#]
   - **Security Profile:** mgmt-security

7. Click **Apply** and then **OK**

### Add Management SSID to Management Bridge

8. Navigate to **Bridge** → **Ports**

9. Click **New**:
    - **Interface:** wlan2
    - **Bridge:** br-mgmt
    - **Comment:** Management Wi-Fi

10. Click **Apply** and then **OK**

### Test Management Wi-Fi

11. On your laptop or phone, look for the **Student[*]** SSID you created in step 6 above.

12. Connect using the password you created.

13. You should receive an IP address in the **10.10.255.x** subnet (from the main router's DHCP server for VLAN 255).

14. Verify you can access **10.10.255.1** (main router). You should also be able to reach the mAP at whatever IP it received from the DHCP server.

---

## Lab 16.8 — Backup mAP Configuration

Before proceeding, back up the mAP configuration.

### Create Binary Backup

1. Navigate to **Files**

2. Click **Backup**

3. Configure:
   - **Name:** mAP-full-config
   - **Password:** [Optional encryption password]

4. Click **Backup**

5. The file appears in the file list. Right-click and **Download** to your laptop.

### Create RSC Export

6. Open **New Terminal**

7. Run:
   ```
   /export file=mAP-export
   ```

8. Navigate to **Files**

9. Download the `mAP-export.rsc` file to your laptop.

> **Why both?** The .backup file is a complete binary backup for restoring to this device. The .rsc file is human-readable and can be used to recreate the configuration on a different device — which leads us to Lab 17 (Scripting).

---

## Lab 16 Summary

Your mAP is now:

- ✅ Reset to clean configuration (no hidden surprises)
- ✅ Fallback mode configured (192.168.89.0/27 with DHCP and Wi-Fi)
- ✅ WAN DHCP client on ether1 (gets internet from any upstream network)
- ✅ Tagged VLANs 20/30/40 carried over trunk when connected to main router
- ✅ WireGuard tunnel configured (works from anywhere with internet)
- ✅ RoMON enabled (manage both devices from one connection)
- ✅ Management Wi-Fi broadcasting on VLAN 255
- ✅ Backed up

**What you can do now:**

| Scenario | What Works |
|----------|------------|
| Connected to main router (normal) | Gets 10.10.255.x via DHCP, full VLAN access via trunk |
| Connected to any internet | Gets WAN IP via DHCP, WireGuard tunnels home automatically |
| Standalone (no connection) | Fallback Wi-Fi + ether2 give local access on 192.168.89.x |
| Need to manage main router | Connect to mAP, use RoMON to reach main router |

---
