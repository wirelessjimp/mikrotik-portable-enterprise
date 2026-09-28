# Appendix B — Reset Procedures

## RouterOS Devices (hEX, RB5009, L009, mAP, hAP)

### Method 1: Reset via WinBox

This is the easiest method when you have access to the device.

1. Connect to the device via WinBox

2. Navigate to **System** → **Reset Configuration**

3. Choose reset options:
   - **Keep User Configuration:** Preserves users and passwords
   - **No Default Configuration:** Boots with blank config (no bridge, no DHCP, no firewall)
   - **Do Not Backup:** Don't save current config before reset
   - **Run After Reset:** Script to run after reset (advanced)

4. Click **Reset Configuration**

5. Click **Yes** to confirm

6. Device reboots with selected configuration

**After reset with "No Default Configuration":**
- Device responds on 192.168.88.1 on all ports
- No DHCP server — set static IP on your laptop (192.168.88.2/24)
- Login: admin, no password

**After reset with default configuration:**
- Device responds on 192.168.88.1
- DHCP server running (192.168.88.10-254)
- Default firewall and NAT configured
- Login: admin, no password

### Method 2: Reset via Terminal

1. Connect to device via WinBox, SSH, or console

2. Run:
   ```
   /system reset-configuration no-defaults=yes skip-backup=yes
   ```

   Or for default configuration:
   ```
   /system reset-configuration
   ```

3. Confirm when prompted

### Method 3: Reset Button (Physical)

Use when you can't access the device via network.

**Reset to default configuration:**

1. Unplug power from the device

2. Press and hold the reset button

3. While holding the button, plug in power

4. Watch the USR LED — it will start flashing

5. Release the button when USR LED starts flashing

6. Device boots with default configuration

**Reset to no configuration (blank):**

1. Follow steps 1-5 above

2. Continue holding until USR LED stops flashing and goes solid

3. Release the button

4. Device boots with no configuration

### Reset Button Timing Reference

| Duration | Action |
|----------|--------|
| Until USR flashes (~5 sec) | Reset to default configuration |
| Until USR solid (~10 sec) | Reset to blank configuration |
| Until USR off (~15 sec) | Enter Netinstall mode |

### Method 4: Netinstall (Complete Reinstall)

Use when the device won't boot or is severely corrupted.

**Requirements:**
- Windows PC
- Netinstall software (download from mikrotik.com)
- Ethernet cable directly to device
- RouterOS package file for your device

**Process:**

1. Download Netinstall from https://mikrotik.com/download

2. Download the RouterOS package for your device architecture

3. Run Netinstall as Administrator

4. Configure your PC's NIC with a static IP (e.g., 192.168.88.2/24)

5. In Netinstall, click **Net booting** and enable it
   - Set boot server IP to your PC's IP

6. Put the device in Netinstall mode:
   - Hold reset button
   - Apply power
   - Hold for ~15 seconds until USR LED turns off
   - Release button

7. Device appears in Netinstall

8. Select the device, browse to the RouterOS package, click **Install**

9. Wait for installation to complete

10. Device reboots with fresh RouterOS

---

## SwOS Devices (CSS/CRS Switches)

SwOS reset works differently than RouterOS.

### Method 1: Reset via Web Interface

1. Access the switch web interface

2. Click **System** tab

3. Scroll to the bottom

4. Click **Reset Configuration**

5. Switch reboots with default settings

### Method 2: Reset Button (Physical)

**Reset to default configuration:**

1. Unplug power from the switch

2. Press and hold the reset button

3. While holding, plug in power

4. Hold for 5 seconds

5. Release the button

6. Switch boots with default configuration

**Default SwOS settings:**
- IP: 192.168.88.1
- DHCP client enabled (will also accept DHCP address)
- Login: admin, no password

---

## Post-Reset Checklist

After any reset, verify:

- [ ] Can access device (WinBox, web, or SSH)
- [ ] Set admin password
- [ ] Set device identity
- [ ] Configure time zone
- [ ] Check for firmware updates
- [ ] Apply your configuration (or import from script)

---

## WinBox Safe Mode

Safe Mode is an "undo" feature for risky configuration changes. If you lose connection while in Safe Mode, the router automatically reverts your changes.

### How It Works

1. Enter Safe Mode — router starts tracking all changes
2. Make your configuration changes
3. If you lose connection, router waits ~9 minutes then reverts everything
4. If everything works, exit Safe Mode to commit the changes

### Using Safe Mode

**Enter Safe Mode:**
- Click the **Safe Mode** button in WinBox title bar, OR
- Press **Ctrl+X** in WinBox

The title bar shows **[Safe Mode]** when active.

**Exit Safe Mode (commit changes):**
- Click **Safe Mode** button again, OR
- Press **Ctrl+X** again

**Cancel changes manually:**
- Press **Ctrl+D** to revert immediately without waiting

### When to Use Safe Mode

- Changing firewall rules on a remote device
- Modifying IP addresses or routes
- Disabling interfaces
- Any change that might lock you out

### Example: Risky Firewall Change

1. Connect to remote router via WinBox
2. Press **Ctrl+X** (enter Safe Mode)
3. Add your new firewall rule
4. Test that you can still access the router
5. If working: Press **Ctrl+X** again (commit)
6. If locked out: Wait ~9 minutes, changes revert automatically

### Important Notes

- Safe Mode timeout is approximately 9 minutes (TCP timeout)
- Only works in WinBox and CLI — not WebFig
- Changes are tracked per-session — if WinBox crashes, changes revert
- Multiple users: only one can be in Safe Mode at a time

> **This is your "oh shit" insurance.** Before making any change that might disconnect you from a remote device, enter Safe Mode first.

---
