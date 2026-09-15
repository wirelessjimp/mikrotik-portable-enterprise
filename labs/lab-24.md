# Lab 24 — WAN Sources

*Prerequisites: Lab 1 (Initial Configuration), Lab 10 (Firewall)*

> **Class note:** Labs 24.3 (Failover) and 24.4 (Wi-Fi Client Mode) can be completed in class using your wired WAN and the R550 Wi-Fi. Labs 24.1 (USB Tethering) and 24.2 (USB Cellular) require personal devices and are reference material for home use.

MikroTik routers aren't limited to a single wired WAN connection. This lab covers alternative WAN sources — cellular, Wi-Fi client mode, USB tethering — and how to configure automatic failover between any two WAN connections.

**Use cases:**
- Portable deployment with cellular primary
- Trade show with venue Wi-Fi as backup
- Home/office with dual ISP failover
- Travel router that connects to hotel Wi-Fi

> **Tested Configuration:** USB cellular was tested with a T-Mobile USB hotspot device. Wi-Fi client mode was proven at MWC Barcelona 2026 using a hAP ax².

---

## Lab 24.1 — USB Tethering (Android)

The simplest cellular WAN: tether your Android phone via USB.

### Connect and Enable

1. Connect your Android phone to the MikroTik's USB port with a USB cable

2. On your phone:
   - Go to **Settings** → **Network & Internet** → **Hotspot & tethering**
   - Enable **USB tethering**

3. On your MikroTik, navigate to **Interfaces**

4. A new interface should appear (typically named **lte1** or similar)

5. Check if the interface is enabled and has a status of "running"

### Configure the Interface

6. If the interface doesn't automatically get an IP:
   - Navigate to **IP** → **DHCP Client**
   - Click **Add New**
   - **Interface:** [Your USB interface]
   - **Add Default Route:** yes
   - Click **OK**

7. Verify the interface received an IP address

### Test Connectivity

8. Navigate to **Tools** → **Ping**

9. Ping **8.8.8.8**

10. If successful, your MikroTik now has internet via your phone

> **Tip: Preserve your USB port.** If your USB port is in use for storage (containers, speedtest server), you can still use your Android phone as a WAN source. Use a USB-C to Ethernet adapter on your phone — the same kind you'd use on a laptop. Plug the Ethernet side into any available MikroTik port. Your phone shares cellular over Ethernet, and the MikroTik sees it as a standard DHCP client on that port — no USB tethering configuration required. Just add a DHCP client on that interface.

---

## Lab 24.2 — USB Cellular Modem

Standalone USB cellular modems (hotspots) often work directly with MikroTik.

### Connect the Modem

1. Insert the USB cellular modem into the MikroTik's USB port

2. Wait for the modem to initialize (may take 30-60 seconds)

3. Navigate to **Interfaces**

4. Look for a new interface (may be named **lte1**, **ppp-out1**, or the modem's name)

### Enable DirectIP Mode (if needed)

Some modems require DirectIP mode:

1. Open **Terminal**

2. Run:
   ```
   /port firmware set ignore-directip-modem=no
   ```

3. Reboot the router:
   ```
   /system reboot
   ```

4. After reboot, check **Interfaces** for the modem

### Configure DHCP Client

5. If the modem interface doesn't auto-configure:
   - Navigate to **IP** → **DHCP Client**
   - Add a new client on the modem interface
   - Set **Add Default Route:** yes

6. Verify IP address assignment

### Test Connectivity

7. Ping external addresses to verify connectivity

> **Tip: Plug-and-play USB cellular sticks.** Some USB cellular modems work with zero configuration on either side. The TCL LINKPORT IK511 (available from T-Mobile, ~$96 or free with plan) plugs directly into the MikroTik's USB port (with a USB-C to USB-A adapter) and starts passing traffic immediately. Note that this does occupy the USB port, so you'll need to choose between cellular and USB storage.

---

## Lab 24.3 — Wi-Fi Client Mode for WAN

Use a MikroTik's Wi-Fi radio to connect to an existing network (hotel, venue, guest Wi-Fi) as a WAN source.

> **Tested Configuration:** This method was proven at MWC Barcelona 2026 using a hAP ax² connecting to venue Wi-Fi. The mAP 2nd works the same way (2.4 GHz only). For dual-band 802.11ax, use a hAP ax² or hAP ax³.

### Understanding the Options

There are two approaches to Wi-Fi client mode:

| Mode | How It Works | Pros | Cons |
|------|--------------|------|------|
| Station Bridge | L2 bridge, passes traffic transparently | Single IP space | Unreliable on wifiwave2 devices |
| Routed NAT Gateway | Separate network, NATs traffic | Reliable, tested | Double NAT |

**We use the Routed NAT Gateway method.** Station-pseudobridge mode on wifiwave2 devices (hAP ax², mAP, etc.) does not reliably pass Layer 2 traffic. The routed approach works consistently.

### Scenario

You want to use your mAP (from Lab 16) to connect to a guest Wi-Fi network and provide that internet connection to your main router.

```
Guest Wi-Fi )))──► mAP (Wi-Fi client) ──► Main Router (ether)
                   NAT gateway              Gets internet
                   172.16.139.1             via mAP
```

### Prepare the mAP

Before reconfiguring the mAP as a Wi-Fi client gateway, back up your current configuration and reset to defaults.

1. In WinBox, connect to the mAP

2. Navigate to **Files**, then click **Backup** in the right-hand panel
   - **Name:** mAP-ap-config
   - Click **Backup**

3. Download the backup file to your computer — select the file and click **Download** in the right-hand panel

4. Navigate to **System** → **Reset Configuration**
   - Leave **No Default Configuration** unchecked — we want factory defaults as our starting point
   - Click **Reset Configuration**

5. The mAP will reboot. Connect your laptop directly to **ether2** on the mAP — after factory reset, ether2 is the LAN port on the default bridge (192.168.88.0/24). Open WinBox and connect to 192.168.88.1

> **Note:** After a factory reset, the admin password is blank. If WinBox has a saved password for this device, clear it out before connecting. On first login, RouterOS will prompt you to set a new password — you can set it back to what you had before.

### Configure mAP as Wi-Fi Client Gateway

#### Create Security Profile for Target Network

1. In WinBox, navigate to **Wireless** → **Security Profiles**

> **Note:** The mAP shows both **Wireless** and **WiFi** in the menu. Use **Wireless** — the WiFi menu is for newer wifiwave2 devices and does not control the mAP's radio.

2. Click **Add New**:
   - **Name:** guest-wifi
   - **Mode:** dynamic keys
   - **Authentication Types:** WPA2 PSK
   - **WPA2 Pre-Shared Key:** [Password for target network]

3. Click **Apply** and **OK**

#### Configure Wi-Fi Interface as Station

4. Navigate to **Wireless** → **WiFi Interfaces**

5. Double-click on **wlan1**

6. Click the **Wireless** tab

7. Configure:
   - **Mode:** station
   - **Band:** 2GHz-only-N (mAP) or 5GHz-a/n/ac (for 5 GHz)
   - **SSID:** [Target network SSID]
   - **Security Profile:** guest-wifi
   - **Country:** [Your country]

7. Click **Apply** and **OK**

8. The interface will attempt to connect. Click the **Status** tab on the interface window — at the bottom of that tab, it should show "connected to ess"

#### Create WAN Bridge

9. Navigate to **Bridge**

10. Click **Add New**:
    - **Name:** br-wan
    - **Comment:** Wi-Fi WAN bridge

11. Click **Apply** and **OK**

12. Click the **Ports** tab

13. Find **wlan1** in the port list — it's already assigned to the default bridge. Double-click to edit:
    - Change **Bridge** from **bridge** to **br-wan**

14. Click **Apply** and **OK**

#### Configure DHCP Client on WAN Bridge

15. Navigate to **IP** → **DHCP Client**

16. Click **Add New**:
    - On the **DHCP** tab:
      - **Interface:** br-wan
      - **Add Default Route:** yes
    - Click the **Advanced** tab:
      - **Default Route Distance:** 1
      - **Check Gateway:** ping

17. Click **Apply** and **OK**

18. Verify the DHCP client receives an IP from the guest network

#### Create LAN Bridge (for downstream devices)

19. Navigate to **Bridge**

20. Click **Add New**:
    - **Name:** br-lan
    - **Comment:** LAN for downstream

21. Click **OK**

22. In the **Bridge** window, click the **Ports** tab, then click **Add New**:
    - **Interface:** ether1 (or ether2, depending on your wiring)
    - **Bridge:** br-lan

23. Click **Apply** and **OK**

#### Configure LAN IP Address

23. Navigate to **IP** → **Addresses**

24. Click **Add New**:
    - **Address:** 172.16.139.1/24
    - **Interface:** br-lan
    - **Comment:** mAP LAN

25. Click **Apply** and **OK**

#### Configure DHCP Server for LAN

26. Navigate to **IP** → **Pool**

27. Click **Add New**:
    - **Name:** lan-pool
    - **Addresses:** 172.16.139.10-172.16.139.250

28. Click **OK**

29. Navigate to **IP** → **DHCP Server**

30. Click **Add New**:
    - **Name:** lan-dhcp
    - **Interface:** br-lan
    - **Address Pool:** lan-pool
    - **Lease Time:** 01:00:00

31. Click **OK**

32. Click the **Networks** tab

33. Click **Add New**:
    - **Address:** 172.16.139.0/24
    - **Gateway:** 172.16.139.1
    - **DNS Servers:** 172.16.139.1

34. Click **OK**

#### Configure NAT (Masquerade)

35. Navigate to **IP** → **Firewall** → **NAT**

36. Click **Add New**:
    - **Chain:** srcnat
    - **Out. Interface:** br-wan
   - Click the **Action** tab:
    - **Action:** masquerade

37. Click **Apply** and **OK**

#### Configure DNS

38. Navigate to **IP** → **DNS**

39. Set:
    - **Allow Remote Requests:** ✓ Checked

40. Click **OK**

### Connect Main Router to mAP

You've just turned the mAP into a Wi-Fi-to-Ethernet gateway. It connects to a Wi-Fi network, gets an IP via DHCP, and shares that connection out its ethernet port with its own DHCP server (172.16.139.0/24). Your main router doesn't know or care that the upstream is Wi-Fi — it just sees another ethernet port handing out DHCP addresses.

This is the same concept as Lab 24.3 where you used a phone as a second WAN. The mAP replaces the phone.

41. Disconnect the mAP's ethernet cable from its current port on your main router

42. Reconnect it to a port that is **not** part of a bridge or VLAN — you need a standalone port. If you completed Lab 24.3, ether3 already has a DHCP client configured and is ready to use

43. If using a port that doesn't have a DHCP client yet, add one:
    - Navigate to **IP** → **DHCP Client**
    - Click **Add New**
    - On the **DHCP** tab:
      - **Interface:** [the port you connected the mAP to]
      - **Add Default Route:** yes
    - Click the **Advanced** tab:
      - **Default Route Distance:** 10
      - **Check Gateway:** ping
    - Click **Apply** and **OK**

44. Verify the DHCP client receives an IP in the 172.16.139.x range — this confirms the mAP is serving as your upstream

### Test the Setup

45. From the main router, ping 8.8.8.8

46. If successful, you now have internet via the mAP's Wi-Fi connection

### Important Notes

- **Double NAT:** This creates a double NAT situation (guest network NATs you, mAP NATs your main router). For most use cases this is fine.

- **RSC exports strip Wi-Fi passwords:** If you export this config as .rsc, the Wi-Fi passphrase won't be included. Use .backup files for full restore, or re-enter the passphrase manually.

- **RoMON won't work:** When wlan1 is in station mode, RoMON cannot use it. You'll need wired access to manage the mAP.

- **Switching back:** To return the mAP to AP mode, change wlan1 mode back to "ap bridge" and reconfigure as needed.

---

## Lab 24 Summary

| Method | Use Case |
|--------|----------|
| USB Tethering | Quick connectivity via phone |
| USB Modem | Dedicated cellular connection |
| Dual WAN Failover | Automatic backup when primary fails |
| Wi-Fi Client Mode | Connect to venue/hotel Wi-Fi as WAN |
| Mode Button Toggle | One-button switch to phone hotspot (future) |

> **Power note:** With a USB power bank or generator, you can run a MikroTik with cellular or Wi-Fi WAN anywhere — remote site, outdoor event, trade show, or emergency deployment.

---


---
