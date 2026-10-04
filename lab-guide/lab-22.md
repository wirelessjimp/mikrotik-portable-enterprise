# Lab 22 — Useful Tools

*Prerequisites: Lab 1 (Initial Configuration)*

MikroTik includes several built-in tools that are useful for network management and troubleshooting.

---

## Lab 22.1 — TFTP Server

When configuring network gear, you often need a TFTP server for firmware transfers. MikroTik has one built in.

### Verify USB Storage

If you completed Lab 5 (Containers), your USB drive is already formatted and mounted.

1. Navigate to **Files**

2. Verify the **usb1** folder exists

3. If not, format your USB drive:
   - Insert a USB flash drive into your MikroTik's USB port
   - Navigate to **System** → **Disks**
   - Select the USB drive
   - Click **Format Drive**
   - Choose **ext4** format
   - Wait for formatting to complete

### Upload Files to USB

4. Navigate to **Files**

5. Find the **usb1** folder

6. Create a small test file on your computer — open a text editor, type `TFTP test file`, and save it as `tftp-test.txt`
  
7. In the Files window, click **Upload** under Actions on the right

8. Select your `tftp-test.txt` file — it uploads to the root of the file system

9. Drag `tftp-test.txt` from the root into the `usb1` folder

> **Note:** For this lab we're using a simple text file to verify TFTP works. In real deployments, this is where you'd place switch firmware images, configuration files, or any other files you need to serve over TFTP.

### Configure TFTP Server

10. Navigate to **IP** → **TFTP**

11. Click **New**:
    - **Enabled:** ✓ Checked
    - **IP Addresses:** Leave blank (allows all clients) or enter a subnet to restrict access
    - **Req. Filename:** The filename clients will request (e.g., `firmware.bin`)
    - **Real Filename:** The actual file path (e.g., `/usb1/tftp-test.txt`)
    - **Allow:** ✓ Checked
    - **Read Only:** ✓ Checked (recommended for security)

9. Click **Apply & OK**

### Test TFTP Transfer

10. From a client device, use a TFTP client to request the file:
    ```
    tftp 10.10.255.1 -c get tftp-test.txt
    ```

11. The file should transfer from the MikroTik's USB storage

> **Use case:** Firmware upgrades for network devices that require TFTP (many switches, APs, and legacy devices).

---

## Lab 22.2 — FTP Server

For more flexible file transfers, enable the FTP server.

### Enable FTP Service

1. Navigate to **IP** → **Services**

2. Double-click **ftp**

3. Configure:
   - **Enabled:** ✓ Checked
   - **Port:** 21
   - **Available From:** Enter allowed subnets (e.g., 10.10.255.0/24) or leave blank for all

4. Click **OK**

### Create FTP User

5. Navigate to **System** → **Users**

6. Click the **Groups** tab

7. Click **New**:
   - **Name:** ftp
   - **Policies:** ftp, read, write

> **Why write access?** Without the `write` policy, you can download files from the router but not upload to it. With write enabled, you can FTP files directly to any path on the router — including `usb1/` — without using the WinBox upload-then-drag workflow. This is especially useful for uploading media files, container configs, and firmware images that are too large for internal storage.

8. Click **Apply & OK**

9. Click the **Users** tab

10. Click **New**:
    - **Name:** ftpuser
    - **Group:** ftp
    - **Password:** [Create a password]
    - **Confirm Password:** [Retype the password]
    - **Allowed Address:** (optional — restrict by IP)

11. Click **OK**

### Connect via FTP

12. From your computer, connect using an FTP client:
    - **Host:** 10.10.255.1 (use the gateway IP for whatever VLAN you're connected to)
    - **Username:** ftpuser
    - **Password:** [Your password]
    - **Port:** 21

> **Security note:** FTP transmits credentials in plain text. Use only on trusted networks, or restrict access via the Available From setting.

> **FTP client options:**
> **Recommended tools for macOS users:**
> - [Transfer](https://www.intuitibits.com/products/transfer/) ($19.99) — runs TFTP, FTP, SFTP, HTTP, and HTTPS servers on your Mac. Built for network admins. Use it when you need your laptop to serve firmware or configs to network gear during initial setup.
> - [Cyberduck](https://cyberduck.io/) (free) — FTP/SFTP client for uploading files TO the MikroTik, since Finder's FTP is read-only.
>
> The MikroTik's built-in TFTP and FTP servers handle the permanent use case — firmware and configs served from the USB drive without needing a laptop connected.> - **macOS:** Open Terminal and type `ftp 10.10.255.1`, or use [Cyberduck](https://cyberduck.io/) (free)
> 
> - **Windows:** Open File Explorer and type `ftp://10.10.255.1` in the address bar, or use [WinSCP](https://winscp.net/) (free)
> - **Browser:** Most modern browsers (Chrome, Edge, Safari) have removed FTP support. Firefox still has limited support but may not work reliably. Use a dedicated FTP client instead.

> **FTP client tips:**
> - **Windows:** File Explorer supports FTP natively with full read/write — type `ftp://10.10.255.1` in the address bar and enter credentials when prompted. Drag and drop works in both directions.
> - **macOS:** Finder's FTP is **read-only** — you can browse and download, but not upload. Use Terminal (`ftp` command), [Cyberduck](https://cyberduck.io/) (free), or any other FTP client for uploading.
> - **Browser:** Chrome, Edge, and Safari have removed FTP support entirely. Firefox has limited read-only support. Use a dedicated client or your OS file manager instead.

---

## Lab 22.3 — Interface Graphing

Monitor interface utilization over time with built-in graphing.

### Enable Interface Graphs

1. Navigate to **Tools** → **Graphing**

2. Click **New**:
   - **Interface:** Select an interface (e.g., ether1 for WAN)
   - **Allow Address:** 0.0.0.0/0 (or restrict to management subnet)

3. Click **Apply & OK**

4. Repeat for other interfaces you want to monitor

### View Graphs

5. Click the **Interface Graphs** tab

6. Double-click on an interface to view its graph

7. Available views:
   - **Daily:** Last 24 hours
   - **Weekly:** Last 7 days
   - **Monthly:** Last 30 days
   - **Yearly:** Last 365 days

8. When finished, you can close the views.

### Real-Time Interface Stats

For real-time (not historical) stats:

1. Navigate to **Interfaces**

2. Double-click an interface

3. Click the **Traffic** section at the bottom to expand it

4. View real-time transmit/receive rates and graphs

---

## Lab 22.4 — Bandwidth Test

Test throughput between MikroTik devices or to a public bandwidth test server.

### Test to a Public Server

1. Navigate to **Tools** → **Bandwidth Test**

2. Configure:
   - **Test To:** `mikrotik.speedtest.alagas.net`
   - **Protocol:** TCP
   - **Direction:** both
   - **Username:** `speedtest`
   - **Password:** `MikroTikSG`

3. Click **Start**

4. View results showing throughput in both directions

> **Note:** This is a community-run server. Availability may vary. TCP is recommended for testing through NAT.

### Test Between Your Own Devices

For a more controlled test, use your own MikroTik devices.

5. On your **L009**, navigate to **Tools** → **BTest Server**:
   - **Enabled:** ✓ Checked
   - **Authenticate:** Unchecked
   - Click **Apply** and **OK**

6. On your **mAP**, navigate to **Tools** → **Bandwidth Test**

7. Configure:
   - **Test To:** [Your L009's IP address]
   - **Protocol:** TCP
   - **Direction:** both

8. Click **Start**

9. Compare the results — testing between your own devices measures the actual link and device performance without internet variables.

> **Note:** The original MikroTik public server (`bandwidth-test.mikrotik.com`) was shut down in 2025. Additional community servers may be available at [btest-rs](https://github.com/manawenuz/btest-rs).

---

## Lab 22 Summary

| Tool | Purpose |
|------|---------|
| TFTP Server | Firmware transfers to network devices |
| FTP Server | General file sharing |
| Interface Graphing | Historical bandwidth monitoring |
| Bandwidth Test | Throughput testing between devices |

---
