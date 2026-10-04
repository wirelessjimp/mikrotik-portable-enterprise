# Lab 14 — WireGuard Clients

*Prerequisites: Lab 12 (WireGuard server configured)*

This lab covers connecting peers to your WireGuard server:
- **13.1** — MikroTik-to-MikroTik (site-to-site)
- **13.2** — Laptop/desktop client (road warrior)

---

## Lab 14.1 — MikroTik-to-MikroTik (Site-to-Site)

> **Prerequisite:** This lab requires a second MikroTik device (such as a mAP). If you haven't set up your second device yet, skip to Lab 14.2 and return to this lab after completing Lab 13.

This scenario connects a second MikroTik (like a mAP) back to your main router. Useful for:
- Remote office connectivity
- Portable lab kit that phones home
- Class scenarios with multiple devices

### On the Remote MikroTik (Client Side)

1. Navigate to **WireGuard** and click **New**:
   - **Name:** wg-home
   - **MTU:** 1420
   - **Listen Port:** 51820

2. Click **Apply** & **OK**

3. Copy the **Public Key** from this interface:

   > **Remote Site Public Key:** ________________________________

4. Navigate to **IP** → **Addresses** and click **New**:
   - **Address:** 10.255.255.2/24
   - **Network:** 10.255.255.0
   - **Interface:** wg-home
   - **Comment:** WireGuard to home

5. Click **Apply** & **OK**

### Add Peer on Remote MikroTik

6. Navigate to **WireGuard** → **Peers** tab

7. Click **Add New** and configure:
   - **Interface:** wg-home
   - **Public Key:** [Server Public Key from Lab 12.1]
   - **Endpoint:** [your DDNS address from Lab 12.4]
   - **Endpoint Port:** 51820
   - **Allowed Address:** 10.255.255.0/24, 10.10.0.0/16
   > Click the **+** button to the right of the first field to add a second field for the second IP range.
   - **Persistent Keepalive:** 00:00:25 (25 seconds)

   > **Note:** Allowed Address defines what traffic goes through the tunnel. Include the VPN subnet (10.255.255.0/24) and your lab networks (10.10.0.0/16).

8. Click **Apply** and **OK**

### Add Peer on Server MikroTik (Your Main Router)

9. On your main router, navigate to **WireGuard** → **Peers**

10. Click **Add New** and configure:
    - **Interface:** wg-server
    - **Public Key:** [Remote Site Public Key from step 3]
    - **Allowed Address:** 10.255.255.2/32
    - **Comment:** Remote MikroTik

    > **Note:** We don't set Endpoint here because the remote site initiates the connection. The server learns the endpoint dynamically.

11. Click **Apply** and **OK**

### Verify Connection

12. On the remote MikroTik, navigate to **WireGuard** → **Peers**

13. Check the **Last Handshake** column — it should show a recent timestamp.

14. Test connectivity:
    ```
    /ping 10.255.255.1
    ```

15. If the ping succeeds, the tunnel is working.

### Add Route for Lab Networks (Remote Side)

For the remote MikroTik to reach your lab networks (10.10.x.x), add a route:

16. Navigate to **IP** → **Routes**

17. Click **Add New**:
    - **Dst. Address:** 10.10.0.0/16
    - **Gateway:** 10.255.255.1
    - **Comment:** Lab networks via WireGuard

18. Click **Apply** and **OK**

---

## Lab 14.2 — Laptop/Desktop Client (Road Warrior)

This scenario lets you connect a laptop or desktop to your network from anywhere using the WireGuard app. For phones and tablets, see Lab 15 (Back to Home) — the MikroTik app makes mobile setup much simpler.

### Install WireGuard on Your Laptop

Before configuring the MikroTik, install the WireGuard client on your laptop:

- **Windows:** Download from https://wireguard.com and install
- **macOS:** Install from the App Store or https://wireguard.com
- **Linux:** Install for your distribution (e.g., `sudo apt install wireguard`)

### Create Peer on the Server

1. On your main router, navigate to **WireGuard** → **Peers**

2. Click **Add New** and configure:
    - **Name:** Laptop Client
    - **Interface:** wg-server
    - **Private Key:** Click the **+** (plus) button next to the field, then select **auto** from the dropdown — this generates a keypair for the client
    - **Allowed Address:** 10.255.255.10/32

3. Scroll down to the **Client** fields and configure:
    - **Client Address:** 10.255.255.10/32
    - **Client DNS:** 10.10.255.1
    - **Client Endpoint:** [your DDNS address from Lab 12.4]
    - **Client Keepalive:** 00:00:25
    - **Client Allowed Address:** Remove `::/0` and add `10.10.0.0/16` and `10.255.255.0/24`
    > Click the **+** button to the right of the first field to add a second field for the second IP range.

4. Click **Apply**

   > **Note:** The **Public Key**, **Client Config**, and **Client QR** fields will be blank until after you click Apply. They populate automatically once the keypair is generated.

5. After applying, two fields at the bottom will populate:
    - **Client Config** — a complete WireGuard configuration file, ready to paste
    - **Client QR** — a QR code for mobile devices

6. Copy the entire contents of the **Client Config** field.

7. Click **OK**

### Configure the WireGuard Client

8. Open the WireGuard app on your laptop.

9. Click **Add Tunnel** → **Add empty tunnel**

10. Replace any existing content with the configuration you copied from step 6.

11. Name the tunnel (e.g., "Lab Router")

12. Save the new tunnel.

13. If you are connected to your router through the backdoor port, switch to a connection that will put you on the WAN side of the router (an existing Wi-Fi connection).

14. In the new tunnel, click on **Activate**.

> **Behind NAT?** If your MikroTik is behind another router (e.g., a lab or office setup), the DDNS endpoint won't work because the WireGuard port isn't forwarded. For local testing, edit the tunnel configuration and change the Endpoint to your MikroTik's local IP address (e.g., `10.22.251.56:51820`). For remote access behind NAT, see Lab 15 (Back to Home) which uses MikroTik's cloud relay to avoid port forwarding.

### Verify Connection

13. With the tunnel active, try to reach your MikroTik:
   - Open browser to `http://10.10.255.1` (or any lab IP)
   - Or ping 10.255.255.1 from terminal

14. On your MikroTik, check **WireGuard** → **Peers** — you should see a recent handshake and traffic counters for the Laptop Client peer.

> **Note:** The QR code generated in step 5 can also be used with the WireGuard mobile app. Open the app, tap **+** → **Scan from QR code**, and point the camera at the QR code displayed on the MikroTik screen. This is the fastest way to set up a phone connection. We'll complete the full setup in Lab 15 next.

---

## Lab 14 Summary

You now have:
- ✅ Site-to-site VPN between two MikroTik devices
- ✅ Road warrior configuration for phones/laptops
- ✅ Full access to your lab networks from anywhere

WireGuard is now your secure tunnel back home.

---
