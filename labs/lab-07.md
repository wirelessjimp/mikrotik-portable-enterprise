# Lab 7 — mAP Enterprise Wi-Fi (RADIUS Test)

*Prerequisites: Lab 5 (User Manager, your test user, and the RADIUS secret in Lab Notes), Lab 6 (the tunnel, and a WinBox window for each device)*

**Why:** You'll broadcast a WPA2-Enterprise network from the mAP. When a phone joins it, the mAP doesn't check the password itself. It asks the User Manager on your L009 to do it, across the tunnel from Lab 6.

Use the window named in each step. The **mAP window** is the one connected to `10.255.255.2`, and the **L009 window** is the one connected to `192.168.88.1`.

### 7.1 Create the security profile

1. **mAP window:** click **Wireless**, then **Wireless**, then the **Security Profiles** tab. Double-click `fallback-security`. The **General** tab shows **WPA2 Pre-Shared Key** filled with dots. Write a throwaway password in **Lab Notes** first, in the **Wi-Fi Passwords** table, on the `Management` row. Then **delete the dots**, type the new password, and click **Apply**, then **OK**.

   > **Why:** `fallback-security` protects your fallback Wi-Fi, your emergency access to the mAP. It came with a password from the instructor's script, so change it to your own.

2. Rename the fallback network and let the radio choose its channel. Click the **WiFi Interfaces** tab, double-click `wlan1`, and click the **Wireless** tab. Set these in the order the window lists them:
   - **Frequency:** `auto`
   - **Scan List:** click **Advanced Mode** (top right of the dialog) first. Click the **+** and enter `2412`, click the **+** again for `2437`, and again for `2462`. Each one goes on its own line.
   - **SSID:** `Student31-Fallback`. Use your own label (`Student01` to `Student12`). Record it in **Lab Notes**, in the **Wi-Fi Passwords** table, on the `Management` row.
   - **Country:** check that it reads `etsi`. If it doesn't, choose `etsi` from the list.

   Click **Apply**, then **OK**. The status line at the bottom briefly says it's searching for a frequency, then goes back to `running ap`.

   > **Why:** Every kit started with the same network name, the same password, and the same channel, so a phone could join your neighbor's mAP, and seven radios would talk over each other. Your own name, your own password, and a channel the radio picks from a short list (1, 6, and 11) fix both. **Country** tells the radio which channels it's allowed to use.

   > **Note:** `wlan2` follows `wlan1`, so phones on `Student31-EAP` move to the new channel with it.

3. Click the **Security Profiles** tab, then **New**.
4. On the **General** tab, set these in the order the window lists them:
   - **Name:** `eap-security`
   - **Mode:** `dynamic keys`
   - **Authentication Types:** check **WPA2 EAP** and leave the other boxes alone
5. Click the **RADIUS** tab and check **EAP Accounting**.

   > **Why:** Accounting is how the mAP tells the L009 that a login started. Without it, your phone can join, but the L009's **Sessions** list stays empty.

6. Click **Apply**. The title changes from `New...` to `eap-security`.
7. Click the **EAP** tab. **EAP Methods** reads `passthrough`. Leave it alone and click **OK**.

   > **Why:** `passthrough` hands the login to the RADIUS server. The mAP doesn't check it. *(Describes how passthrough works in general. Not tested with other methods.)*

### 7.2 Create the enterprise SSID

8. **mAP window:** click **Wireless**, then **Wireless**, then the **WiFi Interfaces** tab, then **New**, then **Virtual**. On the **General** tab, leave **Name** at `wlan2` and **Type** at `Virtual`.

   > ### ⚠️ STOP AND READ
   > The left menu has an item called **WiFi** next to **Wireless**. Don't use it. On the mAP it's empty, because the mAP's radio lives under **Wireless**.
9. Click the **Wireless** tab and set these in the order the window lists them:
   - **Mode:** `ap bridge`
   - **SSID:** `Student31-EAP`. Use your own label (`Student01` to `Student12`).
   - **Master Interface:** `wlan1`
   - **Security Profile:** `eap-security`

   Click **Apply**, then **OK**. `wlan2` appears in the list, indented under `wlan1`.

   > **Why:** A virtual AP is a second SSID on the same radio. `wlan1` keeps broadcasting your fallback network, so your emergency access stays.

10. Open **New Terminal** in the mAP window, paste this, and press **Enter**:

```
/interface/bridge/port/add interface=wlan2 bridge=br-fallback comment="Enterprise Wi-Fi"
```

   > **Why:** A bridge port gives a phone on `wlan2` somewhere to land. `br-fallback` already has a DHCP server, so the phone gets an address from it.

### 7.3 Tell the mAP where to send logins

11. **mAP window:** click **RADIUS** in the left menu, then **New**. Set these in the order the window lists them:
   - **Comment:** `L009 User Manager`
   - **Service:** check **wireless**
   - **Address:** `10.255.255.1`
   - **Secret:** the RADIUS secret from **Lab Notes**

   Leave every other field alone and click **Apply**, then **OK**.

   > **Why:** `10.255.255.1` is your L009's address inside the tunnel. Ports `1812` and `1813` are already what User Manager listens on.

### 7.4 Tell the L009 about the mAP

12. **L009 window:** click **User Manager**, then the **Routers** tab, then **New**. Set these in the order the window lists them:
    - **Name:** `mikrotik-map-tunnel`
    - **Address:** `10.255.255.2`
    - **Shared Secret:** the same RADIUS secret from **Lab Notes**

    Click **Apply**, then **OK**. Leave the `mikrotik-ap` entry alone.

    > **Why:** User Manager only answers devices on its Routers list. Over the tunnel, the mAP sends from `10.255.255.2`, and the `mikrotik-ap` entry only covers `10.10.255.0/24`. Keeping both entries means the mAP also works when it's plugged into your L009.

### 7.5 Test it from a phone

13. On your phone, join `Student31-EAP`. Enter `user2@mikrotik.test` and the password from Lab 5.

    > **Android (older versions):** Set **EAP method** to `PEAP`, enter the identity and password, and set **CA certificate** to **Don't validate**. Leave **Advanced** alone. The phone warns `No certificate specified. Your connection won't be private.` Tap **Connect** anyway. You won't see the certificate prompt below, because this setting skips the check.

    > **Android (newer versions):** Leave **EAP method** at `PEAP`, **Phase 2 authentication** at `MSCHAPV2`, **CA certificate** at `Trust on First Use`, and **Anonymous identity** at `anonymous`. Enter your identity and password, then tap **Connect**.

14. **iPhone and newer Android:** the phone shows a certificate named `radius.mikrotik.test`, issued by `local-ca`. The iPhone marks it **Not Trusted**. Android's **Security certificate** window also lists the validity dates and fingerprints. Tap **Trust**.

    > **Why:** Your phone has never heard of `local-ca`, because your router signed its own certificate in Lab 3. It's the same warning your browser gave you then. In a real network you'd install the CA on the device beforehand, so nobody has to tap through the prompt.

15. **L009 window:** click **User Manager**, then the **Sessions** tab. Rows for `user2@mikrotik.test` appear, with **NAS IP Address** `10.255.255.2`. A session that's still active shows an uptime of `0s` until it ends.
16. **mAP window:** in **New Terminal**, run `/ip/dhcp-server/lease/print`. A lease for your phone appears. The address matches the one in your phone's Wi-Fi settings.

> ### ⚠️ STOP AND READ
> If the phone says it can't join, run `/radius/monitor 0 once` in the mAP's Terminal and look at `requests`, `accepts`, and `timeouts`. If `timeouts` climbs and `accepts` stays at `0`, the L009 isn't answering. Check step 12, especially the address. If `accepts` climbs and the **Sessions** list stays empty, check **EAP Accounting** in step 5.
