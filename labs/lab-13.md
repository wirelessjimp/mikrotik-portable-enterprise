# Lab 13 — The Button Script

*Prerequisites: Lab 12 (the mAP is a Wi-Fi client and `class-wifi` exists). A WinBox window to the mAP.*

**Why:** The mAP's side button can run a script. In this lab one press switches the radio between Wi-Fi client (class Wi-Fi) and access point (your fallback SSID), and it changes everything each mode needs along with it. No WinBox, no Terminal.

> **Note:** On the mAP 2nD the button is registered as the **reset button** (`/system/routerboard/reset-button`). `/system/routerboard/mode-button/print` prints nothing.

### 13.1 Check the device mode

1. **mAP window:** in the Terminal, run:

```
/system/device-mode/print
```

The instructor's mAP showed `mode: advanced`, `container: no`, `routerboard: no`. You don't need to change anything.

> **Note:** Running this command writes a dump file to **Files**. Delete it when you're done.

### 13.2 Turn the button on

2. Run:

```
/system/routerboard/reset-button/print
```

It shows `enabled: no`, `hold-time: 0s..1m`, and an empty `on-event`.

3. Run:

```
/system/routerboard/reset-button/set enabled=yes hold-time=1s..5s
```

RouterOS takes it with no error, even though the device mode says `routerboard: no`.

> ### ⚠️ STOP AND READ
> The old lab says a press under 1 second does nothing and a hold over 5 seconds triggers the factory-reset behavior. Hold the button for about 2 seconds, and never for more than 5.

### 13.3 Create the script

4. **mAP window:** click **System**, then **Scripts**, then **New**. Set **Name** to `toggle-wifi-mode`.
5. Check **Don't Require Permissions**.
6. Paste this into **Source**:

```
:local id [/system/identity/get name]
:local pos [:find $id "-mAP"]
:local label $id
:if ([:typeof $pos] != "nil") do={ :set label [:pick $id 0 $pos] }
:local currentMode [/interface/wireless/get wlan1 mode]
:if ($currentMode = "ap-bridge") do={
    /interface/wireless/disable [find master-interface=wlan1]
    /interface/bridge/port/remove [find interface=wlan1]
    /interface/wireless/set wlan1 mode=station ssid="Enterprise Networking" security-profile=class-wifi scan-list=default
    :log info "Button: Wi-Fi switched to STATION mode"
} else={
    /interface/wireless/set wlan1 mode=ap-bridge ssid=($label . "-Fallback") security-profile=fallback-security scan-list=2412,2437,2462
    :if ([:len [/interface/bridge/port/find interface=wlan1]] = 0) do={
        /interface/bridge/port/add bridge=br-fallback interface=wlan1 comment="Fallback Wi-Fi"
    }
    /interface/wireless/enable [find master-interface=wlan1]
    :log info "Button: Wi-Fi switched to AP mode"
}
```

7. Click **Apply**, then **OK**.

> ### ⚠️ STOP AND READ
> Without **Don't Require Permissions**, the button seems to do nothing. The log gets `script,error executing script ... from sys2 failed, please check it manually (not enough permissions)`, and the script's **run-count** stays `0`. On the instructor's router this happened with two scripts, one made in the Terminal and one made in WinBox.

> **Why:** The script switches the mode, the SSID, the security profile, the bridge membership, and every virtual SSID on the radio (`wlan2`, and `wlan3` if you built the guest SSID in Lab 16) together. The old version only flipped the mode. Pressed in station mode, it would have left the SSID set to `Enterprise Networking` with the class password, so the mAP would have started broadcasting the class network's name. The AP SSID is built from your identity, so `Student31-mAP` becomes `Student31-Fallback`.

### 13.4 Assign it to the button

8. In the Terminal, run:

```
/system/routerboard/reset-button/set on-event=toggle-wifi-mode
/system/routerboard/reset-button/print
```

It shows `enabled: yes`, `hold-time: 1s..5s`, and `on-event: toggle-wifi-mode`.

### 13.5 Press it

9. The radio is a Wi-Fi client after Lab 12, so the first press switches it to an access point. Press and hold the button for about 2 seconds, then release. Wait about 10 seconds.
10. In the Terminal, run:

```
/system/script/print detail where name=toggle-wifi-mode
```

**run-count** is `1`, and **last-started** shows the time you pressed.

11. Run:

```
/interface/wireless/print
/interface/bridge/port/print where interface~"wlan"
```

`wlan1` reads `mode=ap-bridge`, `ssid="Student31-Fallback"`, and `security-profile=fallback-security`. `wlan2` has no **X**. Both `wlan1` and `wlan2` are in `br-fallback`.

12. Press and hold the button for about 2 seconds again. Wait about 20 seconds, then run the same two commands, plus:

```
/ip/dhcp-client/print terse where interface=wlan1
/ip/route/print terse where dst-address=0.0.0.0/0
```

`wlan1` reads `mode=station`, `ssid="Enterprise Networking"`, `security-profile=class-wifi`, and shows **R** (running). `wlan2` has an **X**. Only `wlan2` is left in `br-fallback`. The DHCP client is `bound`, and the `wlan1` default route has the **A** flag again.

13. Press the button once more to go back to access point mode, and wait for the Wi-Fi LED. Join **Student31-EAP** from a phone, as in Lab 7 step 5. The phone joins and gets an address in `192.168.89.0/24` (`192.168.89.50` on the instructor's router). A phone that still has a saved certificate profile from Lab 9 may join on its own as that user.

    **L009 window:** in the Terminal, run:

```
/user-manager/session/print where active
```

    A session for your phone shows `nas-port-id="wlan2"`, `nas-identifier="Student31-mAP"`, and `nas-ip-address=10.255.255.2` (the mAP's tunnel address), with `uptime=0s`.

> **Note:** In access point mode, both `wlan1` and `wlan2` showed the **I** flag in the bridge list. The cause is unknown.

> **Note:** The mAP's Wi-Fi LED goes off for a few seconds during a switch and comes back on when the new mode is up. Wait for it before you check anything.
