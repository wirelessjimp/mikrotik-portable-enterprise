# Lab 13 — Dual WAN (Wi-Fi as a Second Internet Connection)

*Prerequisites: Lab 6 (both devices reachable in WinBox) and Lab 8 (the mAP is on the 15 cm jumper to **ether8** on your L009). Your L009's WAN cable is in **ether1**, and you have the class Wi-Fi password from the slide.*

**Why:** Until now the mAP has been an access point. In this lab it becomes a Wi-Fi client. It joins the class Wi-Fi and hands that connection to your L009 as a second way to the internet. Then you unplug your L009's WAN cable and watch the traffic move to the Wi-Fi path by itself.

> **Note:** This is a proof of concept. The mAP's radio is 2.4 GHz only, and that band is crowded in a room full of kits. A real build would use a device with a 5 GHz radio, such as a hAP ax². The steps are the same.

> **Note:** Wi-Fi isn't the only second internet source. A phone tether or a cellular modem can be one too. Both need the router's USB port, which holds your flash drive in this class, so they aren't covered here. The independent guide's **WAN Sources** lab has them.

> ### ⚠️ STOP AND READ
> Step 3 downloads a backup of the mAP to your laptop. Do it before anything else. On the instructor's mAP, the backup file disappeared from **Files** after a reboot. The copy on your laptop is the one that lasts.

### 13.1 Back up the mAP

1. **mAP window:** click **Files**, then click **Backup** under **Actions**.
2. Set **Name** to `mAP-preDualWAN-config`. Leave the password blank, click **Don't Encrypt**, then click **Backup Config**.
3. Select `mAP-preDualWAN-config.backup` in the list (about 28.6 KiB) and click **Download...** under **Actions** to save it to your laptop.
4. Open your laptop's **Downloads** folder and check that the file is there.

   > **Why:** Make a backup before any major change. You won't restore this one in this lab, but you'd want it if something went wrong. The administrative lab's Git section (Appendix F) covers keeping your configurations under version control in GitHub.

> ### 🔐 Treat the backup like a password
> The backup isn't encrypted, and it holds the mAP's whole configuration. Delete it from your laptop when class ends.

### 13.2 Free the radio

5. **mAP window:** click **New Terminal** and run:

```
/interface/wireless/print
```

You see two entries. `wlan1` is the radio, with `mode=ap-bridge` and SSID `Student31-Fallback`. `wlan2` is `interface-type=virtual` with `master-interface=wlan1` and SSID `Student31-EAP`.

   > **Why:** `wlan2` is a second SSID that rides on `wlan1`'s radio. While it's there, `wlan1` can't become a Wi-Fi client.

6. Turn `wlan2` off:

```
/interface/wireless/disable wlan2
```

Run `/interface/wireless/print` again. `wlan2` now has an **X** in front of it, which means disabled. The `Student31-EAP` SSID is off the air. If you need your phone online, join the class Wi-Fi.

### 13.3 Check that the mAP can see the class Wi-Fi

7. Run:

```
/interface/wireless/scan wlan1 duration=10
```

It prints a table for 10 seconds. Find **Enterprise Networking** and note its **CHANNEL** and **SIG**. On the instructor's router it showed channel `2462` (channel 11) at `-37` dBm. If the table doesn't return to the prompt, press **Q**.

> **Note:** The mAP only has a 2.4 GHz radio. If the class SSID doesn't appear, tell the instructor. Scanning briefly interrupts the fallback SSID.

### 13.4 Create a security profile for the class Wi-Fi

8. **mAP window:** click **Wireless**, then **Wireless**, then the **Security Profiles** tab.

   > **Note:** The mAP's left menu has both **Wireless** and **WiFi**. Use **Wireless**. The **WiFi** item is empty on the mAP.

9. Click **New**. If the dialog opens on another tab, click the **General** tab.
10. Set **Name** to `class-wifi` and **Mode** to `dynamic keys`. Under **Authentication Types**, check **WPA2 PSK**. The list starts empty, so that's the only box to check. In **WPA2 Pre-Shared Key**, clear the field, then type the class Wi-Fi password from the slide. Click **Apply**, then **OK**.

    > **Note:** Type the key in this dialog, not in a Terminal command. On the instructor's router, a Terminal command with the password in it left the prompt waiting for a closing quote. The class key ends in a `$`, and RouterOS reads a `$` in a command as the start of a variable, which is the likely cause. If you ever do put the key in a command, write the `$` as `\$`.

11. In the Terminal, run:

```
/interface/wireless/security-profiles/print terse where name=class-wifi
```

The line shows `authentication-types=wpa2-psk`. RouterOS doesn't display the key.

### 13.5 Make wlan1 a Wi-Fi client

12. **mAP window:** in the **Wireless** window, double-click **wlan1**. Set:
    - **Mode:** `station`
    - **SSID:** `Enterprise Networking`
    - **Security Profile:** `class-wifi`
    - **Radio Name:** your label (for example `Student31`)
    - **Scan List:** `default`

    Click **Apply**, then **OK**.

    > **Why:** The scan list of three channels from Lab 7 only makes sense for an access point choosing a channel. A client should look on every channel for the class Wi-Fi.

13. In the Terminal, run:

```
/interface/wireless/monitor wlan1 once
```

**status** should read `connected-to-ess`, with **ssid** `Enterprise Networking`. Run it a second time a few seconds later. A working link also showed `authenticated-clients: 1`. Press **Q** if the output keeps scrolling.

> ### ⚠️ STOP AND READ
> A wrong key doesn't give an error. The mAP connects, gets thrown out about 5 seconds later, and tries again. **status** flips between `connected-to-ess` and `searching-for-network`, and the log fills with `lost connection, received deauth: authentication not valid (2)`. On the instructor's router, the instructor's access point logged the mAP's MAC address as failing authentication too many times. The fix is to open `class-wifi`, clear **WPA2 Pre-Shared Key**, and type the password again.

### 13.6 Take wlan1 out of the bridge

14. **mAP window:** click **Bridge**, then the **Ports** tab. Select the row with the comment **Fallback Wi-Fi** (interface `wlan1`, bridge `br-fallback`) and click the red minus to remove it.

    > **Why:** A Wi-Fi client can't carry bridged traffic the way an access point can. Its traffic has to be routed. The row showed the **I** (inactive) flag while `wlan1` was connected.

15. In the Terminal, run:

```
/interface/bridge/port/print terse where interface=wlan1
```

It prints nothing.

### 13.7 Get an address on the class Wi-Fi

16. **mAP window:** click **IP**, then **DHCP Client**, then **New**. On the **DHCP** tab, set **Interface** to `wlan1` and **Add Default Route** to `no`. Clear **Use Peer DNS** and **Use Peer NTP**. Click **Apply**, then **OK**.

    > **Why:** You'll set the route deliberately in the next section, so nothing changes under you yet.

17. In the Terminal, run:

```
/ip/dhcp-client/print terse where interface=wlan1
```

**status** reads `bound`. Write down the **address** (for example `172.20.26.246/24`) and the **gateway** (`172.20.26.1`).

> **Note:** If the line carries an **I** flag and the comment `Interface not active`, the radio isn't connected. Go back to step 13.

### 13.8 Add the Wi-Fi default route as a standby

18. **mAP window:** click **IP**, then **DHCP Client**, and double-click the client on `wlan1`. Set **Add Default Route** to `yes`. Click the **Advanced** tab and set **Default Route Distance** to `10`. Click **Apply**, then **OK**.
19. In the Terminal, run:

```
/ip/route/print terse where dst-address=0.0.0.0/0
```

You see two default routes. The one through `10.10.255.1` on `br-mgmt` has distance `1` and the **A** (active) flag. The one through your Wi-Fi gateway on `wlan1` has distance `10` and no **A**.

> **Why:** RouterOS uses the default route with the lowest distance. The other one waits as a standby.

### 13.9 Test the Wi-Fi link by itself

20. In the Terminal, add a temporary route. Use your own gateway from step 17:

```
/ip/route/add dst-address=8.8.8.8/32 gateway=172.20.26.1 comment=temp-test
```

21. Ping from your own `wlan1` address (without the `/24`):

```
/ping 8.8.8.8 src-address=172.20.26.246 count=4
```

All four replies come back, with TTL `114` and 12 to 22 ms on the instructor's router.

> **Note:** Don't test with `/ping 8.8.8.8 interface=wlan1`. On the instructor's router it returned `host unreachable` from the mAP's own address while the Wi-Fi link was fine.

22. Remove the temporary route and check:

```
/ip/route/remove [find comment=temp-test]
/ip/route/print terse where dst-address=8.8.8.8/32
```

The second command prints nothing.

### 13.10 Share the Wi-Fi connection with your L009

23. **mAP window:** click **IP**, then **Firewall**, then the **NAT** tab. Click **New**. Set **Chain** to `srcnat` and **Out. Interface** to `wlan1`. Click the **Action** tab, set **Action** to `masquerade`, and click **Apply**, then **OK**.

    > **Note:** Use the **NAT** tab. If the **Chain** list offers `input`, `forward`, and `output` and no `srcnat`, you're on **Filter Rules**.

24. In the Terminal, run:

```
/ip/firewall/nat/print terse
```

Two rules show. `NAT fallback to management` masquerades out `br-mgmt`, and your new one masquerades out `wlan1`.

25. Click the **Filter Rules** tab, then **New**.
26. Set **Comment** to `Forward L009 to Wi-Fi WAN`.
27. Set **Chain** to `forward`.
28. Set **In. Interface** to `br-mgmt`.
29. Set **Out. Interface** to `wlan1`.
30. Click the **Action** tab and check that **Action** is `accept`.
31. Click **Apply**, then **OK**.
32. Drag the new rule above `Drop all else`. A new rule lands at the bottom, below the drop, where it never matches.

    > **Why:** The mAP's `Drop all else` rule drops forwarded traffic that doesn't arrive on a **LAN**-list interface. `br-mgmt` is in the **WAN** list, so without this rule your L009's traffic would be dropped before it got to the Wi-Fi.

33. In the Terminal, run:

```
/ip/firewall/filter/print terse where chain=forward
```

Your new rule is listed above `Drop all else`.

> **Note:** The number in front of the new rule is higher than the one on `Drop all else`, so the list can read `2`, `4`, `3`. The number is an ID, and the position in the list decides which rule matches first.

### 13.11 Test the whole path from your L009

34. **mAP Terminal:** add the temporary route again, with your own gateway:

```
/ip/route/add dst-address=8.8.8.8/32 gateway=172.20.26.1 comment=temp-test
```

35. **L009 window:** click **New Terminal** in the L009's window (the one titled `Student31`) and run:

```
/ip/route/add dst-address=8.8.8.8/32 gateway=10.10.255.250 comment=temp-test
/ping 8.8.8.8 src-address=10.10.255.1 count=4
```

All four replies come back with TTL `113` and 14 to 20 ms on the instructor's router. That's one lower than the mAP's own test, because your ping now crosses the mAP.

36. Remove both temporary routes, the L009's first and then the mAP's, and check each:

```
/ip/route/remove [find comment=temp-test]
/ip/route/print terse where dst-address=8.8.8.8/32
```

Both checks print nothing.

### 13.12 Make Wi-Fi the mAP's way out

37. **mAP window:** click **IP**, then **DHCP Client**, and double-click `client1` (comment `Management DHCP from L009`). Click the **Advanced** tab and set **Default Route Distance** to `20`. Click **Apply**, then **OK**.

    > **Why:** The mAP's default route points back at your L009. Your L009 is about to send internet traffic to the mAP, so without this change the mAP would send it straight back.

38. In the Terminal, run:

```
/ip/route/print terse where dst-address=0.0.0.0/0
```

The `wlan1` route (distance `10`) now has the **A** flag, and the `br-mgmt` route (distance `20`) is the standby.

### 13.13 Give the L009 a second way out

39. **L009 window:** in the Terminal, run:

```
/ip/route/add dst-address=0.0.0.0/0 gateway=10.10.255.250 distance=10 comment=WAN2-via-mAP
/ip/route/print terse where dst-address=0.0.0.0/0
```

The `ether1` route (distance `1`) has the **A** flag. `WAN2-via-mAP` shows `s` for static, distance `10`, no **A**, and an `immediate-gw` on `vlan255`.

> **Note:** `10.10.255.250` is your mAP's address on the jumper. The instructor's mAP kept it. If yours differs, use the address in **Lab Notes**.

### 13.14 Watch it fail over

40. **L009 window:** start a ping that runs until you stop it:

```
/ping 8.8.8.8
```

Watch the **TTL** column. It reads `114`.

41. Unplug the cable from **ether1** on your L009.
42. A couple of pings time out. Then the replies return with TTL `113`.

    > **Why:** The TTL dropped by one because your traffic now crosses one more router, the mAP. It's the column that tells you which path you're on.

43. Plug the cable back into **ether1**. The TTL returns to `114`. On the instructor's router, no pings were lost on the way back.
44. Press **Ctrl+C** to stop the ping.

Optional: while the cable is out, run this in a second Terminal:

```
/ip/route/print terse where dst-address=0.0.0.0/0
```

`WAN2-via-mAP` has the **A** flag and the `ether1` route is gone.

> **Note:** This works because unplugging the cable drops the link. The L009's WAN client has **Check Gateway** set to `none`, so if the connection upstream died while the cable stayed plugged in, nothing would fail over.

### 13.15 Put the mAP back

You don't restore anything. In Lab 14, a button press switches the radio back to an access point, with its fallback SSID, its security profile, its place in `br-fallback`, and `wlan2` turned on. The profile `class-wifi`, the NAT rule and forward rule for `wlan1`, and the DHCP client on `wlan1` stay on the mAP, and so does the new distance `20` on `client1`. In access point mode the `wlan1` client shows the **I** flag with the comment `Interface not active`, and the mAP's only default route is the one through your L009. The distance `20` has to stay, because in station mode `wlan1` (distance `10`) must win.
