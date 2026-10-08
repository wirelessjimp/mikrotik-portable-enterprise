# Lab 17 — Production Readiness (Draft)

*Prerequisites: Labs 1 to 16, whichever you built, and your **Lab Notes** file filled in.*

**Why:** The class left things open on purpose: shared passwords, exported keys, services switched on, and shares that offered more than you meant. This lab closes them, and shows what to check before you use this gear for real.

> ### ⚠️ STOP AND READ
> This lab is a draft. It was read through and its commands checked where they could be, but not every step has been run on a kit. Don't restrict a service until you have another way in. The backdoor port (**ether7**, at `192.168.88.1`) is your permanent way in, so keep it.

### 17.1 Change every classroom credential

Everything in **Lab Notes** was a throwaway, and the instructor's script gave every kit the same starting values for some of them. Change each one, and record the new value.

| Credential | Where you set it | First set in |
|---|---|---|
| L009 admin password | **System**, **Password** | Lab 1.4 |
| mAP admin password | **System**, **Password** | Lab 6.3 |
| Fallback Wi-Fi key | **Wireless**, **Security Profiles**, `fallback-security` | Lab 7.1 |
| RoMON secret, on both devices | **Tools**, **RoMON** | Lab 8.3 |
| RADIUS secret, in three places: the L009's User Manager router entries `mikrotik-ap` and `mikrotik-map-tunnel`, and the mAP's **RADIUS** client | User Manager and **RADIUS** | Lab 5, Lab 7.3, Lab 7.4 |
| `user2` | User Manager | Lab 5 |
| `ftpwrite` and `ftpread` | **System**, **Users** | Lab 14.6 |
| `smbuser` | **IP**, **SMB**, **Users** | Lab 15.4 |
| Hotspot `admin` | **IP**, **Hotspot**, **Users** | 17.5 below |

1. For each row, set a new password and enter it in **Lab Notes**. Use the dialog in WinBox. Don't type a password into a Terminal command, where it stays in the command history.
2. Save **Lab Notes** (Lab 0, steps 4 and 5).

### 17.2 Delete exported keys and backups

3. **L009 Terminal:** list the files that hold certificates or keys:

```
/file/print where name~"p12|key|crt"
```

On the instructor's router it listed four files, all in the `Certificates` folder, and nothing else.

4. Remove each one you exported, by name. For example, `/file/remove Certificates/user1-client.p12`.

   > **Why:** A `.p12` and a `.key` hold a private key. Anyone who can log in to FTP with the read-only user can download them.

5. Delete the same files from your laptop and from your phones' **Downloads** folders.
6. On both devices, list the backups:

```
/file/print where name~"backup"
```

7. Each device keeps an automatic backup from before its last reset. If it holds an old configuration, delete it. **mAP Terminal:**

```
/file/remove flash/auto-before-reset.backup
```

   **L009 Terminal:** its copy sits in the root of **Files**, not in `flash`:

```
/file/remove auto-before-reset.backup
```

8. Delete `mAP-preDualWAN-config.backup` from your laptop. If you did Lab 18, delete `Student31-l009-before-rsc.backup` and `Student31-mAP-before-rsc.backup` too, and the two export files from 18.3.

> **Note:** Deleting the files doesn't delete the certificates. They stay in the router's certificate store.

### 17.3 Back to Home and the MikroTik app

9. **L009 window:** close the **BTH VPN WireGuard** tab. It shows a key and a QR code that let a device into your network. Don't photograph it.

   > **Note:** Leave the Back to Home tunnel in place. It's meant to be used again: the router updates its address by itself when you plug it in somewhere else, and the phone reconnects. The tunnel's peer is created by the feature itself (it shows the **D** flag, for dynamic), so it isn't yours to remove with `/remove`. To drop the connection, delete the VPN from the phone's settings. The router's `/ip/cloud/back-to-home-users` menu lists the users the app added. Revoking the service can't be undone as a pause: you'd create the connection again from the app, and delete the old peer.

10. In the MikroTik app, **uncheck Keep password** on the login screen, and delete the saved router entries.

### 17.4 Turn off what you don't need

11. **mAP Terminal:** list the services:

```
/ip/service/print
```

`ftp`, `telnet`, `www`, and `api` are enabled, and only the firewall's last rule blocks them. Turn them off, and list again. They show **X**.

```
/ip/service/disable ftp,telnet,www,api
/ip/service/print
```

12. **L009 Terminal:** turn FTP off, which Lab 14 turned on:

```
/ip/service/disable ftp
```

13. Restrict who can log in to the L009. Only your backdoor network and the tunnel get in:

```
/ip/service/set winbox address=192.168.88.0/24,10.255.255.0/24
/ip/service/set ssh address=192.168.88.0/24,10.255.255.0/24
/ip/service/set www-ssl address=192.168.88.0/24,10.255.255.0/24
```

   > **Why:** A phone on the enterprise SSID may be able to reach the L009's login at `10.10.255.1` with the admin password, because the mAP passes its traffic along. After this step the L009 answers only the two networks in the list. It also closes the class Wi-Fi path to the WAN address, which the setup's firewall rule `WAN Access from class Wi-Fi` had opened.

14. Check that it worked. WinBox from your laptop on the backdoor still connects. From a phone on `Student31-EAP`, a login at `10.10.255.1` is refused.

### 17.5 The hotspot `admin` user

15. **L009 window:** click **IP**, then **Hotspot**, then the **Users** tab. Double-click `admin` and set a password. Or remove the user, if you don't need it.

   > **Why:** It was created with no password. While the original login page was in place, that would have let a guest sign in with an empty password.

### 17.6 Check how you reach the router

16. From your laptop on the class Wi-Fi, try WinBox to your L009's WAN address from **Lab Notes**. After 17.4 it times out.
17. From a phone off your network, use Back to Home (Lab 11) and open the MikroTik app at `10.10.255.1`. It still works, because that path comes through the tunnel.

### 17.7 Read the firewall

18. **L009 Terminal:** run:

```
/ip/firewall/filter/print where chain=input
```

Find `WAN Access from class Wi-Fi`, which accepts ports 443 and 8291 from `172.20.26.0/24` on the WAN list. After step 13 the services refuse that network anyway, so the rule no longer lets anyone in. It's harmless, and you can remove it if you'd rather not leave it. The last rule drops everything that doesn't come from the **LAN** list.

19. **mAP Terminal:** run:

```
/ip/firewall/filter/print where chain=input
/interface/list/member/print
```

`br-mgmt` is in the **WAN** list, `br-fallback` is in **LAN**, and `wlan1` is in neither. Read it, and change nothing.

### 17.8 Things to know

- **Enterprise clients share `br-fallback`.** They can reach the mAP's own services until you turn those off. The fuller fix is to put the enterprise SSID on its own bridge.
- **Check Certificate** was turned off for the three container pulls in Lab 4. The router's CRL settings are on.
- **Failover (Lab 12):** the L009's WAN client has **Check Gateway** set to `none`, so failover follows a dropped link only. The route `WAN2-via-mAP` still points at an address the mAP has only while it's on the jumper.
