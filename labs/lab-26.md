# Lab 26 — Dual WAN Failover

*Prerequisites: Lab 25 (any WAN source configured), Lab 10 (Firewall)*

Once you have a second WAN connection from Lab 25, configure your router to automatically fail over when the primary goes down.

## Lab 26.1 — Failover Configuration

Use a backup WAN connection when your primary fails.

### Overview

MikroTik can automatically switch between WAN connections based on availability. The cleanest method uses the DHCP client's **default-route-distance** setting — routes are created automatically with the priority you specify.

This method works for any combination of WAN sources:
- Two wired ISPs (ether1 + ether2)
- Wired + cellular
- Wired + Wi-Fi client (via mAP or hAP)
- Cellular + Wi-Fi client

### How It Works

- Each DHCP client creates a default route with a configurable distance
- Lower distance = higher priority
- When a route becomes unreachable, traffic shifts to the next available route
- When the preferred route recovers, traffic shifts back

### Configure Primary WAN (ether1)

1. Navigate to **IP** → **DHCP Client**

2. Find or create the DHCP client for your primary WAN (ether1)

3. Double-click to edit:
   - On the **DHCP** tab:
     - **Interface:** ether1
     - **Add Default Route:** yes
   - Click the **Advanced** tab:
     - **Default Route Distance:** 1
     - **Check Gateway:** ping

4. Click **Apply** and **OK**

### Configure Backup WAN

5. Find or create the DHCP client for your backup WAN interface

6. Double-click to edit:
   - On the **DHCP** tab:
     - **Interface:** [your backup WAN interface]
     - **Add Default Route:** yes
   - Click the **Advanced** tab:
     - **Default Route Distance:** 10
     - **Check Gateway:** ping

7. Click **Apply** and **OK**

> **Note:** Distance ranges from 0-255. The router always prefers the lowest active distance. Setting backup to 10 instead of 2 leaves room to add additional WAN sources at intermediate priorities later.
>
> **Check Gateway** set to **ping** ensures the router actively checks if the gateway is responding. Without it, failover only happens on physical link loss — if the cable stays plugged in but the upstream is dead (ISP outage, phone loses signal), the router won't fail over.

### CLI Reference

```
/ip dhcp-client set [find interface=ether1] default-route-distance=1
/ip dhcp-client set [find interface=lte1] default-route-distance=2
```

### Verify Routes

8. Navigate to **IP** → **Routes**

9. You should see two default routes (0.0.0.0/0):
   - One via ether1 with distance 1
   - One via cellular with distance 2

10. The route with lower distance (ether1) shows as active

### Test Failover

11. Open **New Terminal** in WinBox and start a continuous ping:
    ```
    /ping 8.8.8.8
    ```

12. Disconnect the primary WAN cable

13. Within a few seconds, the cellular route becomes active

14. Ping 8.8.8.8 — should still work via cellular

15. Reconnect primary WAN

16. Traffic returns to ether1 automatically

> **Note:** Failover time is typically 2-5 seconds. During transition, you may see brief packet loss.

---

## Lab 26.2 — Reset Button Toggle Script

The mAP's physical button (labeled RESET/MODE) can run a script when pressed. This lets you toggle the Wi-Fi radio between AP mode and station mode with a button press — no WinBox or terminal needed.

> **Note:** On the mAP 2nD, this button is registered in RouterOS as the **reset button**, not the mode button. The `/system/routerboard/mode-button` command exists but does not function on this device. Use `/system/routerboard/reset-button` instead.

### Create the Toggle Script

1. Navigate to **System** → **Scripts**

2. Click **New**:
   - **Name:** toggle-wifi-mode
   - **Source:** Paste the following:

```
:local currentMode [/interface/wireless/get wlan1 mode]
:if ($currentMode = "ap-bridge") do={
    /interface/wireless/set wlan1 mode=station
    :log info "Wi-Fi switched to STATION mode"
} else={
    /interface/wireless/set wlan1 mode=ap-bridge
    :log info "Wi-Fi switched to AP mode"
}
```

3. Click **Apply** and **OK**

### Assign the Script to the Button

4. In the terminal, run:

```
/system/routerboard/reset-button/set enabled=yes on-event=toggle-wifi-mode hold-time=1s..5s
```

5. Verify:

```
/system/routerboard/reset-button/print
```

You should see:
```
    enabled: yes
  hold-time: 1s..5s
   on-event: toggle-wifi-mode
```

> **Important:** The `hold-time=1s..5s` setting means:
> - A quick tap (under 1 second) does nothing — prevents accidental toggles
> - A hold between 1 and 5 seconds runs the script
> - A hold longer than 5 seconds triggers the normal factory reset behavior
>
> This protects against both accidental mode changes and accidental resets.

### Test the Toggle

6. Press and hold the button for about 2 seconds, then release

7. Check the wireless mode:

```
/interface/wireless/print detail where name=wlan1
```

Look for `mode=` — it should have changed from `station` to `ap-bridge` or vice versa.

8. Press and hold the button again for 2 seconds to toggle back

9. Verify the mode changed back

### Verify in System History

You can confirm the button presses and script execution in the system history:

```
/system/history/print
```

Each button press shows as a `device changed` action with the trace showing `mode-button/script:toggle-wifi-mode` and the action number.

### Pre-configure the Fallback SSID

For the toggle to be useful, the mAP needs a security profile ready for station mode. Pre-configure a "fallback" profile that connects to a known network — typically your phone's hotspot with a predictable name and password.

10. Navigate to **Wireless** → **Security Profiles**

11. Create a profile (or verify the one from Lab 25.4 exists):
    - **Name:** fallback-hotspot
    - **Mode:** dynamic keys
    - **Authentication Types:** WPA2 PSK
    - **WPA2 Pre-Shared Key:** [Your hotspot password]

12. Click **Apply** and **OK**

When you need emergency internet:
1. Enable your phone's hotspot (with your known SSID/password)
2. Press and hold the button on the mAP for 2 seconds
3. The mAP switches to station mode and connects to your hotspot
4. You now have internet through your phone

### Temporarily Changing the Target Network

The script toggles the mode, but the SSID and security profile stay as last configured. To connect to a different network (venue Wi-Fi, a client's network):

1. Connect to the mAP via WinBox (through ethernet)
2. Navigate to **Wireless** → click on **wlan1** → **Wireless** tab
3. Change the **SSID** to the target network's name
4. Edit the **Security Profile** or change the pre-shared key in the existing profile
5. Press the button to toggle to station mode

When done, change the SSID and password back to your phone's hotspot settings so the button works as expected next time.

> **Tip:** Write down your phone's hotspot SSID and password somewhere permanent. When you're done with a temporary network, restore these values so the one-button toggle always has a known target.

---


## Lab 26 Summary

| Feature | Purpose |
|---------|----------|
| Route Distance | Controls which WAN is preferred (lower = preferred) |
| Check Gateway | Actively verifies gateway is responding |
| Reset Button Script | One-button toggle between AP and station mode |

---
