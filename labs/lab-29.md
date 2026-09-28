# Lab 29 — Administrative Tasks

*Prerequisites: Lab 1 (Initial Configuration)*

Routine administrative tasks you'll perform throughout the life of your MikroTik.

---

## Lab 29.1 — Change Router Identity

1. Navigate to **System** → **Identity**

2. Enter a descriptive name

3. Click **OK**

The identity appears in:
- WinBox title bar
- Terminal prompt
- Neighbor discovery
- SNMP (if configured)

---

## Lab 29.2 — Change Admin Password

1. Navigate to **System** → **Users**

2. Double-click on **admin** (or your admin user)

3. Click **Password** under Actions in the right-hand column

4. Enter:
   - **New Password:** [your new password]
   - **Confirm Password:** [repeat new password]

5. Click **Change Now**

> **Best practice:** Use strong passwords. Consider creating named user accounts instead of using the default "admin" account.

---

## Lab 29.3 — RouterOS Software Upgrade

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

## Lab 29.4 — RouterBOARD Firmware Upgrade

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

## Lab 29.5 — Create Additional Users

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

## Lab 29.6 — Scheduled Reboot

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

## Lab 29.7 — View Logs

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

## Lab 29.9 — Version Control with Git (Optional)

In Lab 6, you learned that RSC exports are plain text scripts you can read, edit, and share. Git takes this further — it tracks every change you make to those exports over time, so you can see exactly what changed, when, and roll back if needed.

### What You Need

- A [GitHub](https://github.com) account (free)
- Git installed on your laptop:
  - **macOS:** Open Terminal and type `git`. If not installed, macOS prompts you to install it.
  - **Windows:** Download from [git-scm.com](https://git-scm.com/)

### Create a Repository

1. On GitHub, click the **+** in the top right and select **New repository**

2. Configure:
   - **Name:** `mikrotik-configs`
   - **Private:** Selected (your configs shouldn't be public)
   - **Add a README:** Checked

3. Click **Create repository**

4. On your laptop, open Terminal and clone it:
   
   ```
   cd ~
   git clone https://github.com/YOUR_USERNAME/mikrotik-configs.git
   cd mikrotik-configs
   ```

### Save Your First Export

5. On your MikroTik, create a compact export:

   ```
   /export compact file=config-export
   ```

6. Download `config-export.rsc` from **Files** to your laptop

7. Copy it into your repo folder:

   ```
   cp ~/Downloads/config-export.rsc ~/mikrotik-configs/main-router.rsc
   ```

8. Commit and push:

   ```
   git add .
   git commit -m "Initial config export"
   git push
   ```

### Track Changes Over Time

9. Make a change on your router (add a firewall rule, change a DHCP setting, anything)

10. Export again:

   ```
   /export compact file=config-export
   ```

11. Download and copy over the previous file:

   ```
   cp ~/Downloads/config-export.rsc ~/mikrotik-configs/main-router.rsc
   ```

12. See what changed:

   ```
   git diff
   ```

 Git shows exactly which lines were added, removed, or modified — highlighted in green and red.

13. Commit the change:

   ```
   git add .
   git commit -m "Added firewall rule for guest network"
   git push
   ```

### View History

14. See all your changes over time:

   ```
   git log --oneline
   ```

15. See what changed in a specific commit:

   ```
   git show COMMIT_HASH
   ```

16. On GitHub, click on your `.rsc` file and click **History** to see every version with diffs.

### Multiple Devices

For multiple devices, save each as a separate file:

   ```
   ~/mikrotik-configs/
   ├── main-router.rsc
   ├── mAP.rsc
   └── class-router.rsc
   ```

Each device's config is tracked independently. One repo, all your devices, full history.

> **Why this matters:** Six months from now, when something breaks and you can't remember what you changed, `git log` and `git diff` tell you exactly what happened. It's the network equivalent of having a changelog for every config change you've ever made.

## Lab 29 Summary

| Task | Location |
|------|----------|
| Change identity | System → Identity |
| Change password | System → Users → [user] → Password |
| Software update | System → Packages → Check For Updates |
| Firmware update | System → RouterBOARD → Upgrade |
| Add users | System → Users |
| Scheduled tasks | System → Scheduler |
| View logs | Log |
| Track changes | Using Git |

---
