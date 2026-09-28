# Lab 12 — WireGuard Server

*Prerequisites: Lab 4 (WAN access configured), Lab 6 (backup completed)*

WireGuard is a modern VPN protocol built into RouterOS v7. It's faster and simpler to configure than IPsec or OpenVPN. This lab configures your MikroTik as a WireGuard server that can accept connections from other MikroTik devices or standard WireGuard clients (laptops, desktops).

### Use Cases

- **Site-to-site:** Connect two MikroTik routers across the internet (e.g., home and office)
- **Road warrior:** Connect your phone or laptop back to your network while traveling
- **Lab access:** Reach your lab environment from anywhere

---

## Lab 12.1 — Create WireGuard Interface

1. Navigate to **WireGuard** in the left menu.

2. Click **New** and configure:
   - **Name:** wg-server (the default name is "wg1" — rename it for clarity)
   - **MTU:** 1420 (default, accounts for WireGuard overhead)
   - **Listen Port:** 51820 (standard WireGuard port; default 13231 also works)

3. Click **Apply** & **OK**

4. **Important:** After creation, the interface displays **Public Key** and **Private Key**. 
   
   Copy the **Public Key** and save it somewhere — remote peers need this to connect.

   > **Note:** Create a new text note on your computer and save the **Public Key** on that note. Make sure you label the note as your WireGuard Public Key (WG_Pub_Key).

---

## Lab 12.2 — Assign IP Address

The WireGuard interface needs an IP address. We'll use a dedicated subnet for VPN clients.

1. Navigate to **IP** → **Addresses**

2. Click **New** and configure:
   - **Comment:** WireGuard VPN
   - **Address:** 10.255.255.1/24
   - **Network:** 10.255.255.0
   - **Interface:** wg-server

3. Click **Apply** & **OK**

> **IP Scheme:** The server is 10.255.255.1. Peers will be assigned addresses like 10.255.255.2, 10.255.255.3, etc.

---

## Lab 12.3 — Firewall Rules

Allow WireGuard traffic from the internet and permit VPN clients to access internal resources.

### Allow WireGuard Port

1. Navigate to **IP** → **Firewall**

2. Click **New** and configure on the **General** tab:
   - **Comment:** `WireGuard from internet`
   - **Chain:** input
   - **Protocol:** udp
   - **Dst. Port:** 51820
   - **In. Interface:** ether1 (WAN)

3. Click the **Action** tab:
   - **Action:** accept

4. Click **Apply** & **OK**

### Allow VPN Client Traffic

6. Click **New** to create a new firewall rule. Configure on the **General** tab:
   - **Enabled:** ✅ Checked
   - **Comment:** `WireGuard clients forward`
   - **Chain:** forward
   - **Src. Address:** 10.255.255.0/24

7. Click the **Action** tab:
   - **Action:** accept

9. Click **Apply** & **OK**

10. Move both rules up so they are positioned after the VLAN 255 rules and before any default drop rules.

### Add WireGuard Interface to LAN List

11. Navigate to **Interfaces** → **Interface List**

12. Click **New**:
   - **List:** LAN
   - **Interface:** wg-server
   - **Comment:** WireGuard VPN

13. Click **Apply** & **OK**

---

## Lab 12.4 — Enable MikroTik Cloud (DDNS)

If your internet connection has a dynamic IP address (most do), you need a way for remote peers to find your router. MikroTik Cloud provides free DDNS.

1. Navigate to **IP** → **Cloud**

2. Configure:
   - **DDNS Enabled:** Checked
   - **DDNS Update Interval:** 01:00:00 (1 hour)
   - **Update Time:** Checked

3. Click **Apply**

4. If no DNS name appears, click **Force Update**.

5. Record your DDNS address:

   > **DDNS Address:** ________________________________.sn.mynetname.net

Remote peers will connect to this address instead of your IP.

> **Note:** If your router is behind another router or NAT device, you'll see a warning: "Router is behind a NAT. Remote connection might not work." WireGuard still works, but you may need to forward the WireGuard listen port (51820) on the upstream device. Lab 15 (Back to Home) avoids this requirement by using MikroTik's cloud relay.
>
> Also, if your router is behind too many NAT connections, it might not be able to update the DNS Name.

6. Click **OK** to close the Cloud window.

---

## Lab 12.5 — Prepare for Peers

At this point, your WireGuard server is ready. To add peers, you'll need:

| Item | Your Value |
|------|------------|
| Server Public Key | (from Lab 12.1) |
| Server Endpoint | [your DDNS address]:51820 |
| Server VPN Subnet | 10.255.255.0/24 |
| Allowed Networks | 10.10.0.0/16 (your lab networks) |

Peers are configured in Lab 14.

---

## Lab 12 Summary

You now have:
- ✅ WireGuard interface created and listening
- ✅ VPN subnet configured (10.255.255.0/24)
- ✅ Firewall rules allowing VPN traffic
- ✅ DDNS configured for dynamic IP handling

Your MikroTik is now a WireGuard VPN server ready to accept connections.

---
