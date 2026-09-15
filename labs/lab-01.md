# Lab 1 — Initial Configuration of the MikroTik Router

*Prerequisites: None*

This lab establishes baseline configuration on a factory-fresh MikroTik router (referred to as "MT" throughout this guide).

---

## 1.1 — Physical Setup and First Login

1. Locate the default credentials. Check two places:
   - **Sticker on the bottom of the router** — Take a clear photo; the printed font is small
   - **Paper guide included in the box** — Often easier to read

2. Power on the router and wait approximately two minutes for boot.

3. Connect your laptop to the **backdoor port** (second-to-last copper port — ether4 on hEX S, ether7 on L009/RB5009) and set your laptop's wired NIC to DHCP.

4. Your laptop will receive an IP address in the **192.168.88.0/24** subnet.

5. Open a browser and navigate to **http://192.168.88.1**

6. Log in using the credentials from step 1.

   > **Note:** MikroTik uses both the letter "O" and the number "0" in passwords, and they look nearly identical. If login fails, check for this.

7. When prompted, create a new password and record it for future reference.

---

## 1.2 — Basic Configuration

8. You are now viewing the **Quick Set** interface. Scroll to the **System** section at the bottom.

9. Change the **Router Identity** to something meaningful (e.g., `Lab-Router-01`).

10. Click **Apply Configuration** in the bottom right corner.

> **IMPORTANT:** Ensure your router identity and password have been changed from defaults before proceeding.

---

## 1.3 — WAN Connection

11. Connect the WAN port (**ether1**) to your upstream network.

12. The WAN port uses DHCP by default. Look in the **Internet** section near the top of the Quick Set page for an IP address to appear. Record this WAN IP address — you'll need it later.

---

## 1.4 — Firmware Update

13. At the top right of the screen, click from **Quick Set** to **Advanced**.

    > **Note:** Quick Set is a simplified dashboard view. Advanced (also known as WebFig) is the full configuration interface. Both are part of the WebUI — you'll switch between them as needed. The URL bar will still show "webfig" even though the UI label says "Advanced."

14. Navigate to **System** → **Packages**, then click **Check For Updates** in the Actions column on the right-hand side.

15. Verify the **Channel** dropdown is set to **stable**.

16. If updates are available, click **Download & Install** in the Actions column on the right-hand side. The router will reboot automatically (takes a few minutes).

    > **Note:** After rebooting, the WebFig login screen may reappear quickly because it's cached in your browser — but the router isn't ready yet. Login attempts will fail until the reboot fully completes. WinBox (Lab 3) handles this better by showing connection status, but we aren't there yet.

17. After reboot, log back in and verify the update completed by checking **System** → **Resources** for the new version number.

---
