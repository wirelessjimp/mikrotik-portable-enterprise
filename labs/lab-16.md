# Lab 16 — Media Center

*Prerequisites: Lab 0 (the portal), Lab 4 (your USB drive shows up as `usb1`), Lab 15 (the FTP write user `ftpwrite`, and `tftp-test.txt` in your `tftp-got` folder). VLC is installed on your laptop for the second half.*

**Why:** SMB is the file sharing that works on Windows, macOS, Linux, iOS, and Android. In this lab you share a folder of media from your USB drive, and you lock it down so only one user can read it. On the way you find out that your L009's setup shares more than you meant it to. Then you stream a test video from the same folder with DLNA, using VLC.

### 16.1 Make the media folder

1. **L009 window:** click **New Terminal** and run:

```
/file/add name=usb1/media type=directory
/file/print where name~"usb1/media"
```

The list shows `usb1/media` as a `directory`.

   > **Note:** Your L009's setup already shares `/usb1/media` as **Lab Media Server**. Until now the folder didn't exist, so the share pointed at nothing.

2. Put a file in the folder with FTP. In a terminal on your laptop, in your `tftp-got` folder, run the command for your system. Type `ftpwrite`'s password at the prompt. **Windows:** use `curl.exe`.

```
curl --user ftpwrite -T tftp-test.txt ftp://192.168.88.1/usb1/media/
```

3. In the L009's Terminal, run:

```
/file/print where name~"usb1/media"
```

The list shows `usb1/media` and `usb1/media/tftp-test.txt`, about `125.2KiB`.

### 16.2 See what a guest can see

4. **macOS:** in a terminal on your laptop, ask the L009 which shares it offers to someone with no login:

```
smbutil view -g //192.168.88.1
```

The list shows two shares: `usb1` and `Lab Media Server`.

> ### ⚠️ STOP AND READ
> `usb1` is your whole USB drive, and the L009 offers it to anyone who connects, with no password. Your L009's setup turned that on. The next section turns it off.

### 16.3 Turn off the whole-drive share

5. **L009 Terminal:** run:

```
/disk/set usb1 smb-sharing=no
/ip/smb/shares/print
```

The list shows `pub` (disabled) and `Lab Media Server`. The `usb1` share is gone.

   > **Why:** The share came from a setting on the disk itself. The checkboxes for automatic SMB sharing under **System**, **Disks**, **Settings** were already off, and they weren't the cause.

6. **macOS:** run `smbutil view -g //192.168.88.1` again. It lists only `Lab Media Server`.

### 16.4 Create an SMB user and lock the share to it

7. **L009 window:** click **IP**, then **SMB**, then **Users** on the right.
8. Select **guest** and click **Disable**.
9. Click **New**. Set **Name** to `smbuser`, enter a throwaway **Password**, and check **Read Only**. Click **Apply**, then **OK**.
10. Record the password in **Lab Notes**, in the **SMB** row.
11. Close the **SMB Users** window to get back to **SMB Settings**.
12. Click **Shares** on the right, and double-click **Lab Media Server**. Set **Valid Users** to `smbuser`. Click **Apply**, then **OK**.
13. In the Terminal, run:

```
/ip/smb/users/print
/ip/smb/shares/print
```

`guest` shows **X** (disabled), and `smbuser` is listed as read-only. `Lab Media Server` points at `/usb1/media`, is read-only, and shows `smbuser` under **Valid Users**.

### 16.5 Connect as smbuser

14. **macOS:** in Finder, press **Cmd+K**, enter `smb://192.168.88.1`, and click **Connect**.
15. Sign in as a **Registered User** with `smbuser` and its password. **Uncheck** the box that remembers the password in your keychain, then click **Connect**. If it asks which share, pick **Lab Media Server**.
16. A Finder window titled **Lab Media Server** opens and lists `tftp-test.txt`. The share also shows under **Locations** in the sidebar.
17. Try to copy a file into the window. macOS refuses, because `smbuser` and the share are read-only.
18. Eject the share with the eject icon next to it in the sidebar.

> **Note:** **Windows** and **Linux:** enter `\\192.168.88.1` in File Explorer, or `smb://192.168.88.1` in a Linux file manager, and sign in as `smbuser`.

### 16.6 Get the test video

19. On the portal, in the **Downloads** card, click **lab-media-test.mp4**. Look near the address bar for **Insecure download blocked**, and click **Keep**.
20. Open your **Downloads** folder. `lab-media-test.mp4` is 20,886,840 bytes (Finder shows about 21 MB), 14 seconds of video with a spoken line. To check it, run the command for your system, in the folder holding the file. **macOS:** `shasum -a 256 lab-media-test.mp4`. **Windows:** `certutil -hashfile lab-media-test.mp4 SHA256`. The fingerprint ends in `c7af3d`. The full value is:

```
5beebb05df16fc5863fe04a9502c89194e21fee7400ef1503ee8bfae10c7af3d
```

### 16.7 Put the video in the media folder

21. In a terminal on your laptop, in the folder holding the video, run the command below. Type `ftpwrite`'s password at the prompt. **Windows:** use `curl.exe`.

```
curl --user ftpwrite -T lab-media-test.mp4 ftp://192.168.88.1/usb1/media/
```

22. **L009 Terminal:** run:

```
/file/print where name~"usb1/media"
```

The list shows `usb1/media/lab-media-test.mp4`, `.mp4 file`, about `19.9MiB`, next to `tftp-test.txt`.

> **Note:** Use `.mp4`, `.mp3`, `.avi`, or `.mkv`. The old lab says the L009 doesn't list `.m4v` files, and the fix is to rename them to `.mp4`. Test video from an iPhone should be saved in the most compatible format (not HEVC).

### 16.8 See what DLNA offers right now

23. **Turn Wi-Fi off on your laptop.** If it's on, VLC may look for media servers on the class Wi-Fi and never find your L009. Your connection through the cable keeps working.
24. Open **VLC**. In the sidebar, under **Local Network**, click **Universal Plug'n'Play**. Wait about a minute.
25. Find `Student31 usb1 media` (your own label in place of `Student31`) and expand it. The tree shows the folders on your USB drive, such as `lost+found`, `nginx-conf`, `nginx-content`, `pull`, and `speedtest-logs`, and your video under `media`.
26. Open `media` and double-click `lab-media-test.mp4`. It plays from your L009, and you hear the spoken line.

> ### ⚠️ STOP AND READ
> Your L009's setup shares the whole USB drive with DLNA, the same way it shared it with SMB. Any media player on the cable side can browse your drive's folders, and play any media file anywhere on it. The next section turns that off.

### 16.9 Serve only the media folder

27. **L009 Terminal:** run:

```
/disk/set usb1 media-sharing=no
/ip/media/print
```

The list is empty. The server that shared the whole drive is gone.

   > **Why:** The disk's own media-sharing setting created that server, as its SMB setting created the SMB share.

28. Add a server for the media folder only:

```
/ip/media/add interface=bridge path=usb1/media friendly-name="Lab Media Server"
/ip/media/print
```

The list shows one entry: **Interface** `bridge`, **Friendly Name** `Lab Media Server`, **Path** `usb1/media`, **Allowed IP** `0.0.0.0`, **Status** `Running`.

29. In VLC, close the app and open it again, click **Universal Plug'n'Play**, and wait about a minute. `Lab Media Server` lists only `lab-media-test.mp4`, 14 seconds long. Double-click it to play.

> **Note:** If you turn Wi-Fi back on, VLC may list servers from other routers on the class network, and may stop showing yours.
