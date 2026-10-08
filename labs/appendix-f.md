# Appendix F — Administrative Cheat Sheet


A reference of where to go to do the routine administrative tasks you'll perform throughout the life of your MikroTik. It is not a lab. Each task works on its own, in any order, and some of them you did earlier in the class. They are here so you can find them again later.

---

## F.1 — Change Router Identity

> **Also in:** Lab 1.3 sets your identity the first time.

1. Navigate to **System** → **Identity**

2. Enter a descriptive name

3. Click **OK**

The identity appears in:
- WinBox title bar
- Terminal prompt
- Neighbor discovery
- SNMP (if configured)

---

## F.2 — Change Admin Password

> **Also in:** Lab 1.4 sets your admin password the first time. Record the new one in **Lab Notes**.

1. Navigate to **System** → **Users**

2. Double-click on **admin** (or your admin user)

3. Click **Password** under Actions in the right-hand column

4. Enter:
   - **New Password:** [your new password]
   - **Confirm Password:** [repeat new password]

5. Click **Change Now**

> **Best practice:** Use strong passwords. Consider creating named user accounts instead of using the default "admin" account.

---

## F.3 — RouterOS Software Upgrade

> ### ⚠️ STOP AND READ
> Don't **Download & Install** an update during class. Your kit is tested on RouterOS `7.24.5`, and the container and User Manager packages from the portal are `7.24.5` builds. Checking for updates is fine.

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

## F.4 — RouterBOARD Firmware Upgrade

> **Note:** Your kit's RouterOS is newer than its RouterBOARD firmware, because the instructor's setup doesn't upgrade the firmware. On the instructor's L009, **Current Firmware** reads `7.20.8` against `7.24.5` under **Upgrade Firmware**. The upgrade needs one reboot.

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

## F.5 — Create Additional Users

> **Also in:** Lab 15.6 creates the FTP groups and users the same way.

Instead of sharing the admin account, create individual user accounts.

### Create a User Group

1. Navigate to **System** → **Users**

2. Click the **Groups** tab

3. Click **New**:
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

6. Click **New**:
   - **Name:** [username]
   - **Group:** operators
   - **Password:** [password]
   - **Allowed Address:** (optional — restrict login by IP)

7. Click **OK**

---

## F.6 — Scheduled Reboot

> **Cleanup:** When you finish trying this, remove the scheduler entry, so your kit doesn't reboot on its own later.

Schedule automatic reboots (useful for stability on long-running devices).

1. Navigate to **System** → **Scheduler**

2. Click **New**:
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

## F.7 — View Logs

> **Also in:** Lab 19 reads the log with `/log/print` to check what an import did.

MikroTik logs system events. Review them regularly and when troubleshooting.

### View Logs

1. Navigate to **Log**

2. Scroll through recent events

3. Use the filter field to search (e.g., "error", "dhcp", "wireless")

### Configure Logging

4. Navigate to **System** → **Logging**

5. View logging rules — what events go where (memory, disk, remote)

6. Click **New** to create custom logging rules:
   - **Topics:** Select event types (e.g., dhcp, wireless, firewall)
   - **Action:** Where to send logs (memory, disk, remote)

---

## F.8 — Version Control with Git (Optional)

> **Also in:** Lab 19 creates an export and downloads it.

In Lab 19, you learned that RSC exports are plain text scripts you can read, edit, and share. Git takes this further — it tracks every change you make to those exports over time, so you can see exactly what changed, when, and roll back if needed.

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

---

## F.9 — Telegram Alerts (Optional)

> ### ⚠️ STOP AND READ
> The alert script in this section holds your bot token. A configuration export can include script text, token and all. Don't commit an export to Git (F.8) after you build the script, or remove the token from the script first.

MikroTik can send alerts directly to your phone via Telegram — no containers, no email servers, no external tools. Combined with Netwatch, your router notifies you the moment something goes down.

### Create a Telegram Bot

1. On your phone, open Telegram and search for **@BotFather**

2. Send `/newbot`

3. Follow the prompts:
   - **Name:** Give it a display name (e.g., `MikroTik Alerts`)
   - **Username:** Give it a unique username ending in `bot` (e.g., `mylab_mikrotik_bot`)

4. BotFather replies with your **bot token** — a long string like `123456789:ABCdefGHIjklMNOpqrSTUvwxYZ`. Save this.

### Get Your Chat ID

5. On your phone, search for your new bot in Telegram and send it any message (e.g., `hello`)

6. On your MikroTik, open **New Terminal** and run:

   ```
   /tool/fetch url="https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates" mode=https output=user as-value
   ```
   
Replace `YOUR_BOT_TOKEN` with the token from step 4.

7. In the output, find `"chat":{"id":` followed by a number. That's your **chat ID**. Save it.

### Test a Message

8. Send a test message from the router:

   ```
   /tool/fetch url="https://api.telegram.org/botYOUR_BOT_TOKEN/sendMessage\?chat_id=YOUR_CHAT_ID&text=Hello from MikroTik" mode=https output=none
   ```
   
Replace both `YOUR_BOT_TOKEN` and `YOUR_CHAT_ID`.

9. Check your phone — you should have a Telegram message from your bot.

### Create an Alert Script

10. Navigate to **System** → **Scripts**

11. Click **New**:
    - **Name:** telegram-alert
    - **Source:**
      ```
      :local botToken "YOUR_BOT_TOKEN"
      :local chatID "YOUR_CHAT_ID"
      :local identity [/system/identity/get name]
      :local message ("$identity: $alertMessage")
      /tool/fetch url="https://api.telegram.org/bot$botToken/sendMessage\?chat_id=$chatID&text=$message" mode=https output=none
      ```
      
12. Click **Apply** and **OK**

### Set Up Netwatch Monitoring

Netwatch pings a host on a schedule and runs scripts when the host goes up or down.

13. Navigate to **Tools** → **Netwatch**

14. Click **New** and configure on the **Host** tab:
    - **Host:** `8.8.8.8` (monitors internet connectivity)
    - **Interval:** `00:01:00` (checks every minute)

15. Click the **Up** tab:
    - **Script:**
      ```
      :global alertMessage "Internet connection restored"
      /system/script/run telegram-alert
      ```
      
16. Click the **Down** tab:
    - **Script:**
       ```
      :global alertMessage "Internet connection DOWN"
      /system/script/run telegram-alert
      ```
       
17. Click **Apply** and **OK**

### Test It

18. Unplug your WAN cable

19. Wait up to one minute — you should get a "Internet connection DOWN" message on Telegram

20. Plug the cable back in — you should get "Internet connection restored"

### More Monitoring Ideas

Add additional Netwatch entries for anything with an IP:
Monitor your container host

```
/tool/netwatch/add host=172.17.0.2 interval=00:01:00
down-script=":global alertMessage "OpenSpeedTest container DOWN"; /system/script/run telegram-alert"
up-script=":global alertMessage "OpenSpeedTest container restored"; /system/script/run telegram-alert"
```

Monitor your mAP

```
/tool/netwatch/add host=10.10.255.249 interval=00:01:00
down-script=":global alertMessage "mAP unreachable"; /system/script/run telegram-alert"
up-script=":global alertMessage "mAP back online"; /system/script/run telegram-alert"
```

> **This replaces email.** Lab 15 mentioned email notifications via scripting — Telegram is faster, easier to set up, and doesn't require an SMTP server. Your phone buzzes the moment something breaks.


## Summary

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
| Monitor system | Using Telegram and the botFather |

---
