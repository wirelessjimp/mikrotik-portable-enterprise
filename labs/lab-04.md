# Lab 4 — Configuring WAN Access

*Prerequisites: Lab 1, Lab 3*

This lab enables secure HTTPS and WinBox management from your home network (or any network on the WAN side of the router). This allows you to manage your lab router without being physically connected to it.

> **Use Case:** This configuration turns your MikroTik into a "Network in a Box" — an isolated lab network you can access and manage from your regular home network. Crash your lab all day long; your family's Netflix won't notice.

---

## Lab 4.1 — Create Firewall Rule for WAN Access

1. Navigate to **IP** → **Firewall** and click **Add New**

2. Configure the rule on the **General** tab:
   - **Chain:** input
   - **Src. Address:** `YOUR_MANAGEMENT_NETWORK/24`
   - **Protocol:** tcp
   - **Dst. Port:** 443,8291
   - **In. Interface List:** WAN

   > **Note:** Replace `YOUR_MANAGEMENT_NETWORK/24` with your actual home network. For example, if your home router assigns addresses like 192.168.1.x, enter `192.168.1.0/24`. Port 443 is for HTTPS web access; port 8291 is for WinBox.

3. Click the **Action** tab and ensure **accept** is selected

4. Click the **Comment** tab (or scroll to Comment field) and enter: `WAN Access`

5. Click **Apply** and **OK**

6. The rule appears at the bottom of the list. Click and drag it up to position **#2** (just after the first default rule).

> **Lab Environment Note:** This rule opens both HTTPS (443) and WinBox (8291) from your home network. This is appropriate for a lab router sitting behind your home firewall. If you're deploying a MikroTik as your primary edge firewall exposed directly to the internet, WinBox access from WAN should be disabled or restricted to VPN-only access — but that's beyond the scope of this guide.

---

## Lab 4.2 — Create SSL Certificate

RouterOS requires a Certificate Authority (CA) before you can sign other certificates. We'll create a local CA first, then create and sign the web management certificate.

### Create the CA Certificate

1. Navigate to **System** → **Certificates** and click **Add New**

2. On the **General** tab, configure:
   - **Name:** `local-ca`
   - **Common Name:** `local-ca`
   - **Days Valid:** 3650

3. Click the **Key Usage** tab and enable:
   - key cert. sign
   - crl sign

4. Click **Apply**

5. Under **Actions** on the right, click **Sign**

6. In the Sign dialog:
   - **CA:** Leave empty (self-signing the CA)
   - Click **Start**

7. Wait for status to show "done", then close the Sign dialog.

8. Click **OK** to close the certificate window.

### Create the Web Management Certificate

9. Click **Add New** to create another certificate

10. On the **General** tab, configure:
    - **Name:** `ssl-web-config`
    - **Common Name:** `ssl-web-config`
    - **Days Valid:** 730

11. Click the **Key Usage** tab and enable:
    - tls server
    - digital signature
    - key encipherment

12. Click **Apply**

13. Under **Actions** on the right, click **Sign**

14. In the Sign dialog:
    - **CA:** Select `local-ca` from the dropdown
    - Click **Start**

15. Wait for status to show "done", then close the Sign dialog.

16. Click **OK** to close the certificate window.

---

## Lab 4.3 — Enable HTTPS Service

17. Navigate to **IP** → **Services**

18. Click on **www-ssl** and configure:
    - **Enabled:** Toggle on
    - **Certificate:** ssl-web-config (select from dropdown)

19. Click **Apply** and **OK**

The HTTPS service should now show as enabled (green) in the services list.

---

## Lab 4.4 — Verify HTTPS Backdoor Access

Before disabling HTTP, verify you can reach the router via HTTPS when connected to the backdoor port:

1. Connect your laptop directly to the backdoor port (ether4 on hEX S, ether7 on L009/RB5009)

2. Navigate to: `https://192.168.88.1`

3. Accept the certificate warning and log in

If this works, you're safe to disable HTTP.

---

## Lab 4.5 — Disable HTTP (Cleanup)

Now that HTTPS is confirmed working via the backdoor:

1. Navigate to **IP** → **Services**

2. Click on **www** (port 80)

3. Click **Disable** or toggle it off

4. Click **Apply** and **OK**

> **Note:** With HTTP disabled, all management access uses HTTPS (port 443). The backdoor port still works — you'll just use `https://` instead of `http://`.

---

## Lab 4.6 — Test WAN Access

To test, you must connect from a network on the WAN side of your router — not plugged directly into the router's LAN ports.

1. Disconnect any wired connection between your laptop and the MikroTik's LAN ports (keep the WAN connected to your upstream network).

2. Connect your laptop to your home Wi-Fi (or any network that connects through the MikroTik's WAN port).

3. Open a browser and navigate to: `https://[YOUR_ROUTER_WAN_IP]`
   
   Use the WAN IP address you recorded in Lab 1.

4. Accept the self-signed certificate warning (expected — you created the certificate)

5. Log in with your credentials

6. Test WinBox: Open WinBox and enter the WAN IP address directly in the **Connect To** field (WinBox neighbor discovery won't work across the WAN — it's Layer 2 only).

If successful, you can now manage your router from your home network via both HTTPS and WinBox.

> **Fallback:** If WAN access fails, connect directly to the backdoor port (second-to-last copper port) for guaranteed access.

---

## What You've Accomplished

At the end of Lab 4, you have:

- ✅ Router configured with custom identity and secure password (Lab 1)
- ✅ USB storage formatted and ready (Lab 2)
- ✅ Container and User Manager packages installed (Lab 2)
- ✅ Time and DNS verified working (Lab 2)
- ✅ WinBox installed and connected (Lab 3)
- ✅ Device mode set to "advanced" for full feature access (Lab 3)
- ✅ CLI fundamentals understood (Lab 3)
- ✅ RoMON configured for future multi-device management (Lab 3)
- ✅ Secure HTTPS and WinBox access from your home network (Lab 4)

Your router is now a fully functional, remotely-manageable lab platform. Everything after this builds on this foundation.

---
