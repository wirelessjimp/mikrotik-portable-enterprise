# Lab 22 — Media Center

*Prerequisites: Lab 1 (Initial Configuration), USB storage attached*

MikroTik can serve media files via DLNA/UPnP and SMB. This is useful for demos, trade show booths, or just having fun with your router.

> **Note:** Media server features work best on higher-end devices (RB5009, L009) with adequate RAM and processing power.

### Why Two Methods?

**DLNA/UPnP** works great for media streaming to Windows, Android, and smart TVs. But Apple dropped UPnP support years ago — it doesn't work on macOS or iOS.

**SMB** provides universal file access that works on every platform: Windows, macOS, Linux, iOS, and Android. If you're sharing files at a trade show or demo where you can't control what devices people bring, SMB ensures everyone can access your content.

---

## Lab 22.1 — DLNA/UPnP Media Server

DLNA allows media players (VLC, Windows Media Player, smart TVs) to discover and play media from your MikroTik.

### Prepare Media Files

1. Ensure your USB drive is formatted and mounted (see Lab 21.1)

2. Create a folder for media:
   - Navigate to **Files**
   - Click **New**
   - Under **Name**, enter: `media`
   - Under **Directory**, click the drop-down and select **usb1**
   - Click **OK**

3. Upload media files (videos, music) to the media folder

> **Important:** The WinBox file upload drops files into the router's internal storage, not directly to the USB drive. If your media file is larger than the available internal storage (128 MB on most devices), the upload will fail. Use FTP instead (configured in Lab 21.2) to upload directly to `/usb1/media/`, or remove the USB drive and copy files from your computer, then reinsert it.

> **macOS Finder limitation:** Finder's built-in FTP support is read-only — you can browse and download files, but not upload. Use Terminal (`ftp` command) or a dedicated FTP client like [Cyberduck](https://cyberduck.io/) for uploading files to the router.

### Enable Media Server

> **Important:** When a USB drive is plugged in, MikroTik automatically creates a dynamic media server on the default bridge that exposes the entire USB drive. To prevent this, disable auto-sharing before creating your own entry:
>
> Navigate to **System** → **Disks**, select the USB drive then select **Settings** in the right hand menu, and uncheck **Auto Media Sharing** and **Auto SMB Sharing**. Or via CLI:
> ```
> /disk/settings/set auto-media-sharing=no auto-smb-sharing=no
> ```
>
> **Known issue (as of RouterOS 7.x):** Disabling auto-media-sharing does not fully remove the dynamic entry — it reappears each time the USB drive is reinserted. Clients may still see the dynamic server in their UPnP browser. With a properly configured firewall (a drop-all rule at the end of the input chain), clients can discover the entry but cannot access any content from it. Only your manually created media server on the correct VLAN interface will serve media. The phantom entry is harmless but cannot currently be hidden.

4. Navigate to **IP** → **Media**

5. Click **New**:
   - **Enabled:** ✓ Checked
   - **Interface:** vlan255
   - **Path:** usb1/media/
   - **Friendly Name:** Lab Media Server

6. Click **Apply & OK**

7. Verify **Status** shows **running**

> **Tip:** MikroTik's DLNA server does not recognize all media formats. M4V files (Apple's MP4 variant) will not appear — rename them to `.mp4`. Stick to common formats: `.mp4`, `.mp3`, `.avi`, `.mkv`.

### Play Media on Clients

**Windows Media Player:**
1. Open Windows Media Player
2. Look under Network locations for "Lab Media Server"
3. Browse and play media files

**VLC (Windows/macOS/Linux):**
1. Open VLC
2. Go to **View** → **Playlist**
3. Under Local Network, click **Universal Plug'n'Play**
4. Find "Lab Media Server" and browse content

**VLC (Android):**
1. Open VLC app
2. Go to **Browse**
3. Under Local Network, find your media server
4. Tap to play

> **Note:** DLNA/UPnP doesn't work well on macOS — use SMB instead (Lab 22.2).

---

## Lab 22.2 — SMB File Sharing

SMB (Server Message Block) provides file sharing that works with Windows, macOS, and Linux.

### Enable SMB

1. Navigate to **IP** → **SMB**

2. Configure:
   - **Enabled:** auto
   - **Domain:** WORKGROUP (or your preferred domain)
   - **Comment:** MikroTik File Share
   - **Interfaces:** all (or select specific interfaces)

3. Click **Apply**

### Create SMB Users

4. Click **Users** on the right side of the SMB settings window

6. The default **guest** user is enabled — select it and click **Disable**

7. Click **New** to create a user:
   - **Name:** mediauser
   - **Password:** [Create a password]
   - **Read Only:** ✓ Checked

8. Click **Apply** and **OK**

### Create Shares

9. Click **Shares** on the right side of the SMB settings window

10. Click **New**:
   - **Name:** Lab Media Server
   - **Directory:** /usb1/media
   - **Read Only:** ✓ Checked
   - **Valid Users:** Select **mediauser** from the drop-down

11. Click **Apply** and **OK**

### Connect from Clients

**Windows:**
1. Open File Explorer
2. In the address bar, enter: `\\` followed by your router's gateway IP (e.g., `\\10.10.20.1`)
3. Enter your SMB username and password when prompted
4. Select the share you created — ignore any other shares that appear

**macOS:**
1. Open Finder
2. Press **Cmd+K** (or Go → Connect to Server)
3. Enter: `smb://` followed by your router's gateway IP for your VLAN (e.g., `smb://10.10.20.1`)
4. Click **Connect**
5. Enter your SMB username and password
6. Select the share you created — ignore any other shares that appear

**Linux:**
1. Open file manager
2. Enter in address bar: `smb://` followed by your router's gateway IP (e.g., `smb://10.10.20.1`)
3. Enter your SMB username and password
4. Select the share you created

---

## Lab 22 Summary

| Method | Best For | Client Support |
|--------|----------|----------------|
| DLNA/UPnP | Media streaming to players | Windows, VLC, Smart TVs, Android |
| SMB | File access, macOS support | Windows, macOS, Linux |

> **Reality check:** While this works for demos and small-scale use, a dedicated NAS or media server is better for serious media serving. But for a trade show booth running a demo video on loop? This gets the job done.

---
