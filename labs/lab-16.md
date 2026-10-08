# Lab 16 — Guest Wi-Fi (Optional)

*Prerequisites: Lab 4 (the nginx container, at `172.17.0.4`), Lab 6 (the tunnel), Lab 7 (you've made a virtual SSID before), Lab 8 (the mAP is on the 15 cm jumper), Lab 14 (the FTP write user `ftpwrite`). The mAP must be on the jumper, not on the class network.*

**Why:** A guest network lets visitors join an open Wi-Fi, and their phone pops up a sign-in page. Here the page sends them straight to a welcome page on your own web server. Your L009's setup already built the guest network and the hotspot. You replace the sign-in page, carry the network to the mAP, and add a guest SSID.

### 16.1 Look at the guest network that's already there

1. **L009 window:** click **New Terminal** and run:

```
/interface/bridge/port/print where bridge=br-guest
/ip/hotspot/print
/ip/hotspot/user/print
/ip/dhcp-server/print where name=dhcp-guest
/file/print where name~"hotspot"
```

`br-guest` holds only `ether6`, which shows the **I** flag because nothing is plugged into it. `hotspot1` runs on `br-guest` with the pool `pool-guest` and the profile `hsprof1`. The hotspot users are `default-trial` and `admin`. `dhcp-guest` serves the network. The `hotspot` folder holds the default pages, including `login.html` at 4,423 bytes.

> ### ⚠️ STOP AND READ
> Your L009's setup created a hotspot user `admin` with no password. The next section replaces the sign-in page, but the user stays. Don't leave a guest network running like this after class.

### 16.2 Replace the sign-in page with a redirect

2. On your laptop, in your `tftp-got` folder, create a file named `login.html`. **macOS:** paste this into a terminal. On another system, use a text editor and save the same text.

```
cat > login.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <title>Redirecting...</title>
    <meta http-equiv="refresh" content="0; url=http://172.17.0.4">
</head>
<body>
    <p>Redirecting to welcome page...</p>
</body>
</html>
EOF
```

The file is 204 bytes. The address `172.17.0.4` is your nginx container from Lab 4.

3. **L009 Terminal:** keep the original page by renaming it:

```
/file/set hotspot/login.html name=hotspot/login.html.bak
```

4. In a terminal on your laptop, upload the new page. Type `ftpwrite`'s password at the prompt. **Windows:** use `curl.exe`.

```
curl --user ftpwrite -T login.html ftp://192.168.88.1/hotspot/
```

5. **L009 Terminal:** run:

```
/file/print where name~"hotspot/login"
/container/print where name=nginx
```

`hotspot/login.html` is 204 bytes, `hotspot/login.html.bak` is 4423 bytes, and the nginx container shows **R** (running).

> **Note:** Guests can reach your containers before signing in because the staged walled garden lists `172.17.0.2`, `172.17.0.3`, and `172.17.0.4`.

### 16.3 Carry the guest network to the mAP

The guest network travels over the jumper as VLAN 50, the way VLANs 20, 30, and 40 do.

6. Check that the mAP is on the jumper (Lab 8.1, steps 1 and 2). The mAP takes about 30 seconds after a cable move before WinBox connects to `10.255.255.2`.
7. **L009 window:** click **Interfaces**, then the **VLAN** tab, then **New**. Set these in the order the window lists them:
   - **Comment:** `Trunk VLAN 50`
   - **Name:** `ether8-vlan50`
   - **VLAN ID:** `50`
   - **Interface:** `ether8`

   Click **Apply**, then **OK**.

8. Click **Bridge**, then the **Ports** tab, then **New**. Set **Comment** to `Trunk to guest`, **Interface** to `ether8-vlan50`, and **Bridge** to `br-guest`. Click **Apply**, then **OK**.
9. In the Terminal, run:

```
/interface/vlan/print where name~"vlan50"
/interface/bridge/port/print where bridge=br-guest
```

The VLAN shows **R** (running), and `br-guest` now lists both `ether6` and `ether8-vlan50`.

10. **mAP window:** click **Interfaces**, then the **VLAN** tab, then **New**. Set **Comment** to `Guest VLAN`, **Name** to `ether1-vlan50`, **VLAN ID** to `50`, and **Interface** to `ether1`. Click **Apply**, then **OK**.
11. Click **Bridge**, then the **Bridge** tab, then **New**. Set **Comment** to `Guest network` and **Name** to `br-guest`. Click **Apply**, then **OK**. The bridge gets no IP address, because your L009 serves DHCP and the hotspot.
12. Click the **Ports** tab, then **New**. Set **Comment** to `Guest VLAN trunk`, **Interface** to `ether1-vlan50`, and **Bridge** to `br-guest`. Click **Apply**, then **OK**.
13. In the Terminal, run:

```
/interface/vlan/print where name~"vlan50"
/interface/bridge/port/print where bridge=br-guest
```

`ether1-vlan50` shows **R**, and `br-guest` lists `ether1-vlan50`.

### 16.4 Create the guest SSID

14. **mAP window:** click **Wireless**, then **Wireless**, then the **WiFi Interfaces** tab, then **New**, then **Virtual**. On the **General** tab, leave **Name** at `wlan3` and **Type** at `Virtual`.

    > **Note:** Use **Wireless**, not **WiFi**. On the mAP, **WiFi** is empty.

15. Click the **Wireless** tab and set these in the order the window lists them:
    - **Mode:** `ap bridge`
    - **SSID:** `Student31-Guest` (use your own label)
    - **Master Interface:** `wlan1`
    - **Security Profile:** `default`

    Click **Apply**, then **OK**. `wlan3` appears under `wlan1`.

    > **Why:** The SSID is open, with no password. The hotspot decides what a guest can reach.

16. Open **New Terminal** and paste this, then press **Enter**:

```
/interface/bridge/port/add interface=wlan3 bridge=br-guest comment="Guest Wi-Fi"
```

17. Run:

```
/interface/wireless/print proplist=name,ssid,master-interface,security-profile where name=wlan3
/interface/bridge/port/print where bridge=br-guest
```

`wlan3` shows `Student31-Guest`, master `wlan1`, and profile `default`. Its bridge port shows the **I** flag, the same as `wlan1` and `wlan2` do in access point mode. The cause is unknown.

> **Note:** The button script from Lab 13 switches every virtual SSID on `wlan1`, so it handles `wlan3` too. In station mode a press turns `wlan2` and `wlan3` off, and in access point mode it turns them back on.

### 16.5 Join from a phone

18. On a phone, join `Student31-Guest`. There's no password.
19. The phone shows an alert that you need to sign in. Open it, or open a browser and go to any plain `http` site. The browser ends on your nginx welcome page, served from your own L009.
20. **L009 Terminal:** run:

```
/ip/dhcp-server/lease/print where server=dhcp-guest
/ip/hotspot/host/print
```

The phone shows in both, with a `10.10.50.x` address (`10.10.50.10` on the instructor's router) and the server `hotspot1`.

### 16.6 Test what a guest can reach

21. With the phone still on `Student31-Guest`, open each address in its browser:
    - `http://172.17.0.2:3000`: your OpenSpeedTest container loads, with no sign-in.
    - `http://172.17.0.4`: the nginx welcome page.
    - `https://example.com`: you're sent to the nginx welcome page, and the outside site doesn't load.

    > **Why:** The walled garden lists your three container addresses, so guests reach those without signing in. The hotspot redirects everything else.

    > **Note:** `172.17.0.3` is the iperf3 container. It isn't a web page.
