# Lab 21 — Useful Tools

*Prerequisites: Lab 1 (Initial Configuration)*

MikroTik includes several built-in tools that are useful for network management and troubleshooting.

---

## Lab 21.1 — TFTP Server

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

6. Drag and drop files from your computer into the usb1 folder (or use the Upload button)

### Configure TFTP Server

7. Navigate to **IP** → **TFTP**

8. Click **Add New**:
    - **Enabled:** ✓ Checked
    - **IP Addresses:** Leave blank (allows all clients) or enter a subnet to restrict access
    - **Req. Filename:** The filename clients will request (e.g., `firmware.bin`)
    - **Real Filename:** The actual file path (e.g., `/usb1/actual-firmware-file.bin`)
    - **Allow:** ✓ Checked
    - **Read Only:** ✓ Checked (recommended for security)

9. Click **OK**

### Test TFTP Transfer

10. From a client device, use a TFTP client to request the file:
    ```
    tftp 10.10.255.1 -c get firmware.bin
    ```

11. The file should transfer from the MikroTik's USB storage

> **Use case:** Firmware upgrades for network devices that require TFTP (many switches, APs, and legacy devices).

---

## Lab 21.2 — FTP Server

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

7. Click **Add New**:
   - **Name:** ftp
   - **Policies:** ftp, read (add write if uploads needed)

8. Click **OK**

9. Click the **Users** tab

10. Click **Add New**:
    - **Name:** ftpuser
    - **Group:** ftp
    - **Password:** [Create a password]
    - **Allowed Address:** (optional — restrict by IP)

11. Click **OK**

### Connect via FTP

12. From your computer, connect using any FTP client:
    - **Host:** 10.10.255.1
    - **Username:** ftpuser
    - **Password:** [Your password]
    - **Port:** 21

13. You can also use a browser: `ftp://10.10.255.1` (enter credentials when prompted)

> **Security note:** FTP transmits credentials in plain text. Use only on trusted networks, or restrict access via the Available From setting.

---

## Lab 21.3 — Interface Graphing

Monitor interface utilization over time with built-in graphing.

### Enable Interface Graphs

1. Navigate to **Tools** → **Graphing**

2. Click **Add New**:
   - **Interface:** Select an interface (e.g., ether1 for WAN)
   - **Allow Address:** 0.0.0.0/0 (or restrict to management subnet)

3. Click **OK**

4. Repeat for other interfaces you want to monitor

### View Graphs

5. Click the **Interface Graphs** tab

6. Click on an interface to view its graph

7. Available views:
   - **Daily:** Last 24 hours
   - **Weekly:** Last 7 days
   - **Monthly:** Last 30 days
   - **Yearly:** Last 365 days

### Real-Time Interface Stats

For real-time (not historical) stats:

1. Navigate to **Interfaces**

2. Double-click an interface

3. Click the **Traffic** section at the bottom to expand it

4. View real-time transmit/receive rates and graphs

---

## Lab 21.4 — Bandwidth Test

<!-- TODO: Test Bandwidth Test tool — verify usefulness compared to iperf3 container from Lab 5. Questions: Is bandwidth-test.mikrotik.com still active? Any gotchas with authentication or firewall rules? Does this add value beyond iperf3? -->

Test throughput between MikroTik devices or to the bandwidth-test.mikrotik.com server.

### Test to MikroTik's Public Server

1. Navigate to **Tools** → **Bandwidth Test**

2. Configure:
   - **Test To:** bandwidth-test.mikrotik.com
   - **Protocol:** TCP or UDP
   - **Direction:** both (transmit and receive)
   - **Username:** (leave blank for public server)
   - **Password:** (leave blank for public server)

3. Click **Start**

4. View results showing throughput in both directions

### Test Between Two MikroTik Devices

To test throughput between your main router and the mAP:

**On the device acting as server:**

1. Navigate to **Tools** → **Bandwidth Server**

2. Ensure **Enabled** is checked

3. Note the authentication settings (or disable for testing)

**On the device acting as client:**

4. Navigate to **Tools** → **Bandwidth Test**

5. Configure:
   - **Test To:** [Server device's IP address]
   - **Protocol:** TCP
   - **Direction:** both
   - **Username/Password:** (if authentication enabled on server)

6. Click **Start**

> **Use case:** Verify throughput between sites, test WireGuard tunnel performance, validate switch/cable capacity.

---

## Lab 21 Summary

| Tool | Purpose |
|------|---------|
| TFTP Server | Firmware transfers to network devices |
| FTP Server | General file sharing |
| Interface Graphing | Historical bandwidth monitoring |
| Bandwidth Test | Throughput testing between devices |

---
