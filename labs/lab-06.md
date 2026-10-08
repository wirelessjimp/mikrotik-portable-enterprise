# Lab 6 — Set Up the mAP

*Prerequisites: Lab 1 (L009 identity, password, WAN address recorded in Lab Notes)*

**Why:** The mAP is your second device. It's a small travel router. In this lab you get into it, name it, secure it, check its firmware, and build a WireGuard tunnel between it and your L009. When the mAP moves to the class network, the tunnel is how you still reach it.

> **Note:** The mAP is `mipsbe` with 64 MiB of RAM. That's why it's a basic travel router in this class.

### 6.1 Cable the mAP and log in

1. Check that the L009's AC power cord is plugged in. The mAP gets power from the L009 over PoE, so it won't boot without it.
2. Plug the 15 cm jumper into **ether8** on the L009 and into **ETH1** on the mAP. The mAP boots in about 30 seconds.
3. Move your 1 meter cable from **ether7** on the L009 to **ETH2** on the mAP.

   > **Why:** ETH1 is the mAP's WAN port, and MikroTik doesn't advertise on its WAN port by default, so WinBox wouldn't find it there. ETH2 is the mAP's backdoor.

4. In WinBox, click **Disconnect**, then **Refresh** on the right of the **Neighbors** list. Find the row with Identity **Student-mAP** and IP **192.168.89.1**, and click its **IP address**.
5. WinBox fills in your L009 password. **Clear the Password field**, type `password`, and click **Connect**. **Login** stays `admin`.

   > **Note:** You'll still see **Class-Router** twice. That's the instructor's router.

### 6.2 Set the identity

6. Click **System**, then **Identity**. Replace `Student-mAP` with your label plus `-mAP`, for example `Student31-mAP`. Use your own label (`Student01` to `Student12`). Click **Apply** and **OK**.
7. The title bar and the bottom status bar show the new name.
8. Enter the identity in **Lab Notes**, in the **mAP** column of the **Identity** row.

### 6.3 Set the password

9. Click **System**, then **Password**. Enter the old password (`password`). For **New Password** and **Confirm Password**, use **the same password as your L009**. Record it in **Lab Notes** first, in the **mAP** column of the **Admin Password** row. Click **Change**.

   > **Why:** One password for both devices makes it easier to switch between them.

10. WinBox stays connected after the change. Log out and back in anyway. Click **Disconnect**, find your mAP in **Neighbors** by its new identity, and click its **IP address**. Clear the **Password** field, type the new password, and click **Connect**.

    > **Why:** Logging in again with the new password saves it in WinBox, so you won't have to type it in other places.

### 6.4 Check the firmware

11. Click **System**, then **Resources**. Record **Version** in the **RouterOS Version** row and **Architecture Name** in the **Architecture** row, both in the **mAP** column. The version should be `7.24.5`. The architecture is `mipsbe`, not `arm`. Ignore **Minimum Version**.
12. If **Version** isn't `7.24.5`, click **System**, then **Packages**, click **Stable**, then **Download & Install**. The mAP downloads, installs, and reboots. It takes about four minutes, because the mAP is slow at this.

    > **Note:** A green flag at the top of WinBox appears when an update is available.

### 6.5 Find the mAP's WAN address and test the internet

13. Click **IP**, then **Addresses**. Find the row with the **D** flag and **br-mgmt** as the interface. It's an address in `10.10.x.x`. Record it in **Lab Notes**, in the **mAP** column of the **WAN IP Address** row.

    > **Note:** This is the mAP's address on the L009's side. It changes when the mAP moves to the class network, and you won't need it after that.

14. Click **New Terminal**, paste this, and press **Enter**:

```
/ping 1.1.1.1 count=4
```

15. The summary reads `sent=4 received=4 packet-loss=0%`. On the instructor's router it took about 15 ms.

### 6.6 Allow WinBox on the mAP's WAN side

The mAP's firewall already blocks everything that doesn't come from its backdoor. Your next steps connect to it by its WAN address, so you add one narrow rule.

16. In the mAP's **New Terminal**, run `/ip/firewall/filter/print`. Note the rule with the comment `Drop all else` in the `input` chain. It's rule 3.
17. Paste this command and press **Enter**:

```
/ip/firewall/filter/add chain=input action=accept protocol=tcp dst-port=8291 in-interface=br-mgmt src-address=192.168.88.0/24 comment="Allow WinBox from L009 backdoor" place-before=3
```

18. Run `/ip/firewall/filter/print` again. Your new rule is rule 3, directly above `Drop all else`.

    > **Why:** Rules are read top to bottom, and the first match wins. A rule below `Drop all else` would never be reached. This rule allows only WinBox (port 8291), only on `br-mgmt`, and only from `192.168.88.0/24`, the network your laptop is on when it's plugged into **ether7**. Everything else stays blocked.

    > ### ⚠️ STOP AND READ
    > Don't open the mAP to the whole internet to make WinBox work. A narrow rule costs you one command. A wide-open WinBox port is how routers get taken over.

### 6.7 Open a WinBox session for both devices

19. Click **Disconnect**. Move your 1 meter cable from **ETH2** on the mAP to **ether7** on the L009. Click **Refresh** in **Neighbors**.
20. The mAP won't be listed. In **Connect to**, clear the old `192.168.89.1` and type the mAP's WAN address from **Lab Notes**. Clear **Password**, type your new password, and click **Connect**. The title bar shows `(Student31-mAP) mAP WinBox` with the `10.10.x.x` address.
21. Click the cube icon in the top right, between **Disconnect** and the gear. Its tooltip reads **Open new WinBox app window**. A new WinBox login window opens.
22. In **Neighbors**, find your L009 and click its **IP address** (`192.168.88.1`). Clear **Connect to** if it still shows the mAP's address. Clear **Password**, type your password, and click **Connect**. You now have two WinBox windows open, one for the L009 and one for the mAP. The window titles show `L009UiGS WinBox` and `mAP WinBox`.

> **Why:** You can now copy values between the two devices.

### 6.8 Build the tunnel

Use **New Terminal** in the window named in each step. Copy and paste keys. Don't type them, because a capital I and a lowercase l look alike.

23. **mAP window:** create the tunnel interface and show its public key.

```
/interface/wireguard/add name=wg-home comment="Tunnel to L009"
/interface/wireguard/print proplist=name,public-key
```

24. **L009 window:** show the L009's public key, port, and tunnel address.

```
/interface/wireguard/print proplist=name,public-key,listen-port where name=wg-server
/ip/address/print where interface=wg-server
```

25. **mAP window:** give the tunnel an address and add the L009 as a peer. Replace `<L009-PUBLIC-KEY>` with the key from step 24 and `<L009-WAN-IP>` with your L009's WAN address from **Lab Notes**.

```
/ip/address/add address=10.255.255.2/24 interface=wg-home comment="WireGuard tunnel"
/interface/wireguard/peers/add interface=wg-home name=l009 public-key="<L009-PUBLIC-KEY>" endpoint-address=<L009-WAN-IP> endpoint-port=51820 allowed-address=10.255.255.0/24 persistent-keepalive=25s comment="L009 wg-server"
```

26. **L009 window:** add the mAP as a peer, and let your laptop's traffic reach the mAP through the tunnel. Replace `<MAP-PUBLIC-KEY>` with the key from step 23, and use your own label in the comment.

```
/interface/wireguard/peers/add interface=wg-server name=map public-key="<MAP-PUBLIC-KEY>" allowed-address=10.255.255.2/32 comment="Student31-mAP"
/ip/firewall/nat/add chain=srcnat action=masquerade out-interface=wg-server comment="Masquerade to mAP tunnel"
```

27. **mAP window:** wait about 10 seconds, then run this:

```
/ping 10.255.255.1 count=4
```

    The summary reads `sent=4 received=4 packet-loss=0%`. On the instructor's router it took about 1 ms, because the mAP is still on the jumper.

28. **mAP window:** let the L009 reach WinBox through the tunnel. Run `/ip/firewall/filter/print` and check the number of the `Drop all else` rule. If it's 4, paste this:

```
/ip/firewall/filter/add chain=input action=accept protocol=tcp dst-port=8291 in-interface=wg-home src-address=10.255.255.1 comment="Allow WinBox from L009 over tunnel" place-before=4
```

    If the number isn't 4, replace the `4` at the end with it. Run `/ip/firewall/filter/print` again. The new rule sits directly above `Drop all else`.

29. Click the cube icon for a third window. In **Connect to**, type `10.255.255.2`. Clear **Password**, type your password, and click **Connect**. The title bar shows `admin@10.255.255.2 (Student31-mAP)`.

    > **Why:** `10.255.255.2` is the mAP's address inside the tunnel. It doesn't change when the mAP moves.

### 6.9 Move the mAP to the class network

30. Close any WinBox windows that are connected to the mAP.
31. Unplug the 15 cm jumper from the mAP's **ETH1**. Plug the cable labeled **mAP** into **ETH1**. This cable carries PoE from the class switch, so the mAP reboots.
32. In the L009 window, click **WireGuard**, then the **Peers** tab. Find the `map` peer. Scroll right and watch **Last Handshake**, **Current Endpoint**, **Rx**, and **Tx**. When the handshake appears, the tunnel is up.
33. Open a new WinBox window, connect to `10.255.255.2` as in step 29, and check that it still works.

> ### ⚠️ STOP AND READ
> Your connection to the mAP's old WAN address drops when the mAP moves, and you don't need it again. The tunnel address `10.255.255.2` is how you reach the mAP from here on. If the tunnel doesn't come up, move the cable back to **ETH2** on the mAP and log in at `192.168.89.1`.
