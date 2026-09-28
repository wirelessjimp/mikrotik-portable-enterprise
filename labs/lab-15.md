# Lab 15 — Back to Home VPN

*Prerequisites: Lab 12 (WireGuard server), or can be done standalone*

Back to Home (BTH) is MikroTik's simplified VPN feature. It uses WireGuard under the hood but handles all the key management and relay infrastructure automatically. Unlike the manual WireGuard setup in Labs 12-13, BTH routes all traffic through MikroTik's cloud relay — no port forwarding required, even when the router is behind NAT.

BTH is ideal for:
- Quick phone or laptop connections without manual key exchange
- Demonstrating that VPN doesn't have to be complicated
- Scenarios where the router is behind NAT and direct WireGuard isn't practical

> **Note:** Back to Home requires MikroTik Cloud (DDNS) to be enabled. If you completed Lab 12.4, you're already set.

---

## Lab 15.1 — Enable Back to Home

1. Navigate to **IP** → **Cloud**

2. Click the **BTH VPN** tab across the top of the Cloud window.

3. Set **Back To Home VPN** to **enabled**.

4. Click **Apply**

5. Wait a few seconds, then verify the following fields have populated:
   - **VPN Status:** running
   - **VPN DNS Name:** [your router's BTH address, ending in `.vpn.mynetname.net`]
   - **VPN Port:** [a dynamically assigned port number]

   > **Note:** The VPN DNS Name for BTH (`.vpn.mynetname.net`) is different from your DDNS address (`.sn.mynetname.net`) from Lab 12. BTH uses MikroTik's relay infrastructure rather than connecting directly to your router's public IP.

6. You will also see relay status fields showing which MikroTik relay servers your router has connected to. At least one relay should show as reachable.

   > **Behind NAT?** If your router is behind another router, you'll see a warning at the bottom of the window: "Router is behind a NAT. Remote connection might not work." BTH is specifically designed to work through NAT using the relay — this warning can be safely ignored for BTH connections.

---

## Lab 15.2 — Connect a Phone or Tablet

The simplest way to connect a mobile device is via QR code using the MikroTik app.

1. On your router, click the **BTH VPN WireGuard** tab across the top of the Cloud window.

2. The **VPN WireGuard Client Config** field shows a complete WireGuard configuration, and the **VPN WireGuard Client Config QRCode** is displayed below it.

3. Install the **MikroTik Back To Home** app on your phone or tablet (iOS App Store or Google Play).
   
   <img width="102" height="96" alt="image" src="https://github.com/user-attachments/assets/a98b63d0-ffef-472e-93a1-a1ce6c5b5112" />

5. Open the app and tap **Join shared** --> **Scan QR code**.

6. Tap **Scan QR code** and allow the app to access your camera.

7. Point your camera at the QR code displayed on your router screen.

8. Go ahead and give the tunnel a name you will recognize.

9. The tunnel configures automatically. Your phone is now connected to your network via BTH VPN.

   > **Note:** The BTH client configuration includes two peers — one relay peer and one server peer — and routes all traffic through the VPN (full tunnel). This is different from the manual WireGuard setup in Lab 14, which uses a split tunnel that only routes lab network traffic.

---

## Lab 15.3 — Connect a Laptop or Desktop

For laptops and desktops using the standard WireGuard client:

1. Navigate to **IP** → **Cloud** → **BTH VPN WireGuard** tab.

2. Copy the entire contents of the **VPN WireGuard Client Config** field.

3. Open the WireGuard app on your laptop:
   - **Windows/macOS:** Click **Add Tunnel** → **Add empty tunnel**
   - **Linux:** Create a new configuration file

4. Paste the copied configuration into the tunnel.

5. Name the tunnel (e.g., "MikroTik BTH") and save.

6. Activate the tunnel.

> **Note:** Each device that connects via BTH uses the same client config. For multiple simultaneous connections or per-device configs, use the manual WireGuard peer setup from Lab 14.2 instead.

---

## Lab 15.4 — Verify Connection

1. With the tunnel active, try to reach your MikroTik:
   - Browse to your router's internal IP (e.g., `http://10.10.255.1`)
   - Or ping an internal address from terminal

2. On your MikroTik, navigate to **WireGuard** → **Peers** — the BTH-generated peer will appear in the list alongside any manually configured peers. Check for a recent **Last Handshake** timestamp and non-zero Rx/Tx counters.

---

## Lab 15.5 — Back to Home vs Manual WireGuard

| Feature | Back to Home | Manual WireGuard |
|---------|--------------|------------------|
| Setup complexity | Minimal — QR code or config paste | More steps, manual key exchange |
| Key management | Automatic | Manual |
| Tunnel type | Full tunnel (all traffic) | Split tunnel (lab networks only) |
| NAT traversal | Built-in via relay | Requires port forwarding |
| Multiple peers | Single shared config | Per-device configs, unlimited peers |
| Customization | None | Full control |
| Best for | Quick access, demos, NAT situations | Production, multi-site, specific routing |

**When to use Back to Home:**
- You need quick remote access without port forwarding
- You're behind NAT and direct WireGuard won't reach your router
- You're demonstrating VPN to non-technical users

**When to use manual WireGuard:**
- Multiple devices needing separate peer configs
- Split tunneling (only route specific subnets)
- Site-to-site connectivity
- Integration with existing infrastructure

---

## Lab 15 Summary

You now have:
- ✅ Back to Home VPN enabled and running
- ✅ QR code connection for mobile devices via the MikroTik app
- ✅ Client config for laptop and desktop WireGuard clients
- ✅ Understanding of when to use BTH vs manual WireGuard

Back to Home proves that VPN doesn't have to be complicated — and it works even when your router is behind NAT.

---
