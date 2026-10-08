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

> **Note:** Unless you check **Do Not Backup**, a reset saves the old configuration as `auto-before-reset.backup`. That file holds your old passwords and keys, unencrypted. Delete it when you no longer need it.

**After reset with "No Default Configuration":**
- The device has no IP address and no DHCP server. Login is `admin` with no password.
- Find it by its MAC address. In WinBox, click **Disconnect**, then **Refresh** in **Neighbors**, and click the MAC address, not an IP. WinBox fills in your old password, so clear the **Password** field before you click **Connect**.
- Give it an address or load a configuration script. Appendix A does this for the class kit.

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

`skip-backup=yes` is the same as **Do Not Backup**. Leave it out if you want the safety copy.

### Method 3: Reset Button (Physical)

Use when you can't access the device via network. The button gives you the **default configuration**, not a blank one. For a blank configuration use Method 1 or 2.

**Reset to the default configuration:**

1. Unplug power from the device

2. Press and hold the reset button

3. While holding the button, plug in power

4. Watch the USR LED (or the LED your model's documentation names). When it starts flashing, release the button

5. Device boots with the default configuration

**Don't hold on past the flashing.** Keep holding and the device does something else:

### Reset Button Timing Reference

| Release the button when the LED is... | What happens |
|---------------------------------------|--------------|
| Flashing | Reset to the default configuration |
| Solid (about 5 seconds later) | CAPs mode: the device looks for a CAPsMAN controller. This is not a reset. |
| Off (about 5 seconds after that) | Netinstall mode: the device looks for a Netinstall server |

Holding the button before you apply power, and releasing it about 3 seconds after, loads the backup boot loader.

MikroTik's page on this: https://manual.mikrotik.com/docs/getting-started/configuration-management/routeros-configuration-reset

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
   - Keep holding until the USR LED has gone from flashing to solid to off (about 10 seconds after it starts flashing)
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
- [ ] Apply your configuration (or import from script). For the class kit, load the completed file as Appendix A describes.

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
- Click the **Safe Mode** switch in the WinBox top bar (in WinBox 4 it sits to the right of the undo and redo arrows), OR
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
