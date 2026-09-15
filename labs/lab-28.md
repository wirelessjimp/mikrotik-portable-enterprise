# Lab 28 — Administrative Tasks

*Prerequisites: Lab 1 (Initial Configuration)*

Routine administrative tasks you'll perform throughout the life of your MikroTik.

---

## Lab 28.1 — Change Router Identity

1. Navigate to **System** → **Identity**

2. Enter a descriptive name

3. Click **OK**

The identity appears in:
- WinBox title bar
- Terminal prompt
- Neighbor discovery
- SNMP (if configured)

---

## Lab 28.2 — Change Admin Password

1. Navigate to **System** → **Users**

2. Double-click on **admin** (or your admin user)

3. Click **Password** under Actions in the right-hand column

4. Enter:
   - **New Password:** [your new password]
   - **Confirm Password:** [repeat new password]

5. Click **Change Now**

> **Best practice:** Use strong passwords. Consider creating named user accounts instead of using the default "admin" account.

---

## Lab 28.3 — RouterOS Software Upgrade

MikroTik releases regular updates. Keep your device current for security and features.

### Check for Updates

1. Navigate to **System** → **Packages**

2. Click **Check For Updates**

3. Select **Channel:**
   - **stable:** Production-ready (recommended)
   - **long-term:** Conservative updates, extended support
   - **testing:** Pre-release features
   - **development:** Bleeding edge (not recommended for production)

4. If updates are available, the window will show your **Installed Version** and the **Latest Version** available on that channel

### Install Updates

5. Click **Download & Install**

6. The router will download the update and reboot automatically

7. After reboot, reconnect and verify the new version in **System** → **Packages**

---

## Lab 28.4 — RouterBOARD Firmware Upgrade

Separate from RouterOS, the RouterBOARD firmware is the low-level hardware firmware.

### Check Firmware Version

1. Navigate to **System** → **RouterBOARD**

2. Compare:
   - **Current Firmware:** What's running now
   - **Upgrade Firmware:** What's available

### Upgrade Firmware

3. If versions differ, click **Upgrade** under Actions

4. Click **OK** to confirm

5. The firmware stages for next boot — you must reboot to apply

6. Navigate to **System** → **Reboot**

7. Click **Yes** to reboot

8. After reboot, verify firmware version matches

---

## Lab 28.5 — Create Additional Users

Instead of sharing the admin account, create individual user accounts.

### Create a User Group

1. Navigate to **System** → **Users**

2. Click the **Groups** tab

3. Click **Add New**:
   - **Name:** operators
   - **Policies:** Check the permissions appropriate for this group:
     - **local:** Local console login
     - **telnet:** Telnet access
     - **ssh:** SSH access
     - **ftp:** FTP access
     - **reboot:** Reboot device
     - **read:** View configuration
     - **write:** Modify configuration
     - **policy:** Manage users and groups (leave off for non-admins)
     - **test:** Run tests like ping, traceroute, bandwidth-test
     - **winbox:** WinBox access
     - **password:** Change own password
     - **web:** WebFig access
     - **sniff:** Packet sniffer access
     - **sensitive:** View sensitive info like passwords and keys
     - **api / rest-api:** API access
     - **romon:** RoMON access

> **Tip:** For a basic operator account, start with: local, read, winbox, reboot, and test. Add write only if they need to make changes.

4. Click **Apply** and **OK**

### Create a User

5. Click the **Users** tab

6. Click **Add New**:
   - **Name:** [username]
   - **Group:** operators
   - **Password:** [password]
   - **Allowed Address:** (optional — restrict login by IP)

7. Click **OK**

---

## Lab 28.6 — Scheduled Reboot

Schedule automatic reboots (useful for stability on long-running devices).

1. Navigate to **System** → **Scheduler**

2. Click **Add New**:
   - **Name:** weekly-reboot
   - **Start Date:** [pick a date]
   - **Start Time:** 04:00:00 (or preferred time)
   - **Interval:** 7d 00:00:00 (weekly)
   - **On Event:**
     ```
     /system reboot
     ```

3. Click **OK**

> **Note:** Scheduled reboots can mask underlying issues. Use sparingly and investigate if you need them for stability.

---

## Lab 28.7 — View Logs

MikroTik logs system events. Review them regularly and when troubleshooting.

### View Logs

1. Navigate to **Log**

2. Scroll through recent events

3. Use the filter field to search (e.g., "error", "dhcp", "wireless")

### Configure Logging

4. Navigate to **System** → **Logging**

5. View logging rules — what events go where (memory, disk, remote)

6. Click **Add New** to create custom logging rules:
   - **Topics:** Select event types (e.g., dhcp, wireless, firewall)
   - **Action:** Where to send logs (memory, disk, remote)

---

## Lab 28 Summary

| Task | Location |
|------|----------|
| Change identity | System → Identity |
| Change password | System → Users → [user] → Password |
| Software update | System → Packages → Check For Updates |
| Firmware update | System → RouterBOARD → Upgrade |
| Add users | System → Users |
| Scheduled tasks | System → Scheduler |
| View logs | Log |

---


---

# Lab Notes — Labs 20-26

**Lab 20 — Packet Capture & Torch**

| Item | Value |
|------|-------|
| Streaming PC IP | |
| Streaming Port | 37008 |

**Lab 21 — Useful Tools**

| Item | Value |
|------|-------|
| TFTP Files Location | /usb1/ |
| FTP Username | |
| FTP Password | |

**Lab 22 — Media Center**

| Item | Value |
|------|-------|
| Media Folder Path | /usb1/media/ |
| DLNA Server Name | |
| SMB Username | |
| SMB Password | |

**Lab 23 — NTP**

| Item | Value |
|------|-------|
| NTP Servers | time1.google.com, time2.google.com |
| Local NTP Stratum | 5 |

**Lab 24 — WAN Options & Failover**

| Item | Value |
|------|-------|
| Primary WAN Interface | |
| Primary WAN Distance | 1 |
| Backup WAN Interface | |
| Backup WAN Distance | 2 |
| Cellular Interface Name | |
| Wi-Fi Client SSID | |
| Wi-Fi Client Gateway IP | 172.16.139.1 |

**Lab 26 — Hotspot**

| Item | Value |
|------|-------|
| Guest Bridge Name | |
| Hotspot Interface | |
| Guest VLAN | |

**Lab 28 — Administrative Tasks**

| Item | Value |
|------|-------|
| Router Identity | |
| Admin Username | |
| Additional Users Created | |

---

*Document Version: Draft 1.0*
*Last Updated: March 2026*
