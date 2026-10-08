# Appendix A — If You Fall Behind (Reset and Load a Completed File)

*Prerequisites: none. Use this only if you're far behind and want to skip ahead. It erases the device you reset.*

**Why:** The instructor posts two completed files near the end of Day 2, one for the L009 and one for the mAP. Each one is a script that rebuilds that device as if you'd finished the labs. Secrets and certificates aren't in the scripts, so you add those by hand afterwards. You reset the device first, so the script starts from an empty configuration.

> ### ⚠️ STOP AND READ
> A reset erases everything on the device: its passwords, its certificates, and its container entries. Your **Lab Notes** file on your laptop isn't affected. Reset and load the L009 and the mAP one at a time.

### A.1 Before you reset (both devices)

1. On the portal, download `class-complete-l009.rsc` and `class-complete-map.rsc`. Look near the address bar for **Insecure download blocked**, and click **Keep**.

   > **Why:** Once your laptop is plugged into a reset router, the portal may stop loading by name. Download both files first.

2. Open your **Downloads** folder and check that both files are there.
3. Save your **Lab Notes** (Lab 0, steps 4 and 5), so your records survive.

### A.2 Reset the L009 and load its file

4. **L009 window:** click **System**, then **Reset Configuration**. Check **No Default Configuration**, click **Reset Configuration**, and confirm. The L009 reboots, and your WinBox window drops.

   > **Note:** A reset keeps the installed packages, the device mode, and the USB drive's format. Those were set up once, before class.

5. Wait about a minute. In WinBox, click **Disconnect**, then **Refresh** on the right of the **Neighbors** list. Your L009 shows with no IP address. Click its **MAC address**, not an IP.
6. Leave **Login** as `admin`. The reset removed the password, and WinBox fills in the old one, so **clear the Password field** and leave it empty. Click **Connect**.
7. Click **Files**, then use the upload button to upload `class-complete-l009.rsc`.
8. Click **New Terminal** and run:

```
/import file-name=class-complete-l009.rsc
```

It should print `Script file loaded and executed successfully`. If it prints an error instead, stop and tell the instructor.

   > **Note:** WinBox may drop during the import, because the script puts **ether7** into a bridge. If it does, wait a few seconds and connect to `192.168.88.1`.

9. Check for problems:

```
/log/print where message~"class-complete-l009"
```

You should see a `start` line and a `done` line. A line with `warning` in it names a step that failed. Tell the instructor.

10. Set your identity as in Lab 1.3 and your password as in Lab 1.4. Record both in **Lab Notes**.

### A.3 Reset the mAP and load its file

11. Move your 1 meter cable to **ETH2** on the mAP, as in Lab 6.1.
12. **mAP window:** click **System**, then **Reset Configuration**. Check **No Default Configuration**, click **Reset Configuration**, and confirm. The mAP reboots.
13. Wait about a minute. In WinBox, click **Disconnect**, then **Refresh** in **Neighbors**. Find the mAP by its **MAC address** and click it. Leave **Login** as `admin`, clear the **Password** field, and click **Connect**.
14. **Set the identity first.** Click **System**, then **Identity**. Set it to your label plus `-mAP`, for example `Student31-mAP`, and click **Apply** and **OK**.

    > **Why:** The script builds your SSIDs from the mAP's identity. With the default identity they'd get the wrong label.

15. Click **Files** and upload `class-complete-map.rsc`. Then, in the Terminal, run:

```
/import file-name=class-complete-map.rsc
/log/print where message~"class-complete"
```

The import should print `Script file loaded and executed successfully`. On a reset mAP you should see few or no `exists or failed` lines. Tell the instructor about any `warning` line, and about any `exists or failed` line you can't explain.

16. Reconnect through **Neighbors** at `192.168.89.1`. Set the password as in Lab 6.3, with the same password as your L009.

> **Note:** The guest Wi-Fi from Lab 16 reaches your L009 only while the mAP is on the jumper cable to the L009's **ether8**. On the class network, guests get no address.

> ### ⚠️ STOP AND READ
> The files hold placeholder values, and anyone who downloads the files knows them. Until you change them, anyone can join `Student31-Fallback` with the placeholder key `CHANGE-ME-FALLBACK`, and anyone can log in as `user2@mikrotik.test` with `CHANGE-ME-USER2` once your enterprise Wi-Fi works. The RADIUS secrets (`CHANGE-ME-RADIUS`) are placeholders too. So are the passwords of the L009's FTP users `ftpwrite` and `ftpread` (`CHANGE-ME-FTPWRITE`, `CHANGE-ME-FTPREAD`) and its SMB user `smbuser` (`CHANGE-ME-SMB`). Change all of them in the next section. Don't expect the enterprise Wi-Fi to work until you've done the by-hand steps, including the tunnel.

### A.4 What you still do by hand

**L009**
- Create your certificates (Lab 3).
- Add the three containers, which pulls their images (Lab 4).
- Set your own password on `user2@mikrotik.test` in User Manager (Lab 5).
- Set your own RADIUS secret on the `mikrotik-map-tunnel` router entry (Lab 7.4).
- Add the WireGuard peer for your mAP, with the mAP's public key (Lab 6.8).
- After your certificates exist, turn on HTTPS and give User Manager its certificate (Lab 3.4 and Lab 5). The script skips both until the certificates exist.
- Set your own passwords on `ftpwrite` and `ftpread` (Lab 14.6) and on `smbuser` (Lab 15.4).
- Download `tftp-test.txt` and `lab-media-test.mp4` from the portal and put them on the USB drive (Labs 14 and 15). The TFTP and DLNA settings are already there.
- Optional: replace the hotspot sign-in page with the redirect (Lab 16.2). The guest network is already set up, and until you replace the page, the hotspot's own sign-in form shows.

**mAP**
- Set your own fallback Wi-Fi key (the fallback password step in Lab 7).
- Set the class Wi-Fi key in the `class-wifi` profile (Lab 12.4).
- Set your own RADIUS secret, the same one as on the L009 (Lab 7.3).
- Build the WireGuard tunnel (Lab 6.8).

Record each new password in **Lab Notes**.
