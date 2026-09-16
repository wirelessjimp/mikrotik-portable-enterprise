# Lab 3 — Management Options

*Prerequisites: Lab 1, Lab 2*

MikroTik offers multiple management interfaces. This lab introduces each one and establishes the foundation for managing your router effectively.

---

## Lab 3.1 — WebUI (WebFig and Quick Set)

You've been using the WebUI since Lab 1. Let's clarify what you're looking at.

The MikroTik WebUI has two views:

- **Quick Set** — Simplified dashboard for common settings (WAN, LAN, Wi-Fi basics)
- **Advanced (WebFig)** — Full configuration interface mirroring WinBox functionality

Switch between them using the links in the upper right corner of the page.

**The Terminal button** in the upper right of WebFig opens a CLI session directly in your browser — useful when you need to run commands without switching to WinBox.

> **Note:** The WebFig interface was significantly updated in RouterOS 7.19 to mirror WinBox more closely. If you see screenshots online that look different, they may be from older versions.

---

## Lab 3.2 — WinBox

WinBox is a native application providing the most complete management experience. You downloaded it in Lab 2.

### Install and Launch

1. Locate the WinBox file you downloaded:
   - **Windows:** `winbox64.exe` (run directly, no install needed)
   - **macOS:** `WinBox.dmg` (open and drag to Applications)

2. Launch WinBox.

3. Accept any security warnings from your operating system.

### Connect to Your Router

4. In the **Neighbors** window on the right, your router should appear with its identity.

5. Click on your router to select it.

6. Notice in the left panel that WinBox populates either the **MAC Address** or **IP Address** depending on which column you clicked in the Neighbors list. Clicking on the MAC address column fills in the MAC; clicking on the IP address column fills in the IP.

   > **Lifesaver:** WinBox can connect via MAC address even when the router's IP configuration is broken. If you ever misconfigure IP addressing and lose connectivity, WinBox neighbor discovery via MAC can still get you in.

7. Enter your credentials (user: `admin`, password: what you set in Lab 1).

8. Click **Connect**.

### WinBox Advantages

- **MAC address discovery** — Connect even when IP is misconfigured
- **Automatic reconnection** — Maintains session across router reboots
- **Neighbor discovery** — See all MikroTik devices on the network
- **RoMON access** — Manage remote devices through intermediate MikroTiks (covered in Lab 3.5)
- **Multiple router connections** — Manage several devices simultaneously (may appear as tabs on Windows or separate windows on macOS)

---

## Lab 3.3 — Device Mode

Factory-fresh MikroTik devices (including the hEX S Refresh and hAP ax²) ship in "home" mode, which disables many advanced features including RoMON, containers, hotspot, scheduler, and more. We need to switch to "advanced" mode to unlock the router's full capabilities.

> **Why now?** You just installed WinBox, which handles reboots gracefully by showing connection status and automatically reconnecting. This is a good time to experience that advantage over the WebUI.

### Check Current Mode

1. In WinBox, open a **New Terminal** from the left menu.

2. Run:

   ```
   /system/device-mode/print
   ```

3. Look at the `mode:` line. If it says `home`, continue with this lab. If it says `advanced`, you can skip to Lab 3.4.

### Switch to Advanced Mode

4. Run:

   ```
   /system/device-mode/update mode=advanced
   ```

5. The router will prompt you to confirm by pressing the **mode or reset button** within 5 minutes.

6. Press and hold the reset button firmly until the port LEDs change, then release.

7. The router reboots. Watch WinBox — it will show "Disconnected" then automatically reconnect when the router is ready.

8. After reconnecting, verify the change:

   ```
   /system/device-mode/print
   ```

   Confirm `mode: advanced` is now shown.

> **What this unlocks:** RoMON, containers, hotspot, scheduler, sniffer, bandwidth-test, fetch, and other features that are disabled in "home" mode.

---

## Lab 3.4 — CLI Fundamentals

The command-line interface is essential for:
- Following forum answers and documentation (typically written in CLI syntax)
- Scripting and automation
- Features only accessible via command line
- Faster configuration once you know the commands

### Access the CLI

- **WebUI:** Click **Terminal** in the upper right
- **WinBox:** Click **New Terminal** in the left menu

On first launch, it asks if you want to see the license. Enter `n` to skip.

### Navigation Basics

The CLI is organized in a hierarchy similar to a filesystem:

| Command | Action |
|---------|--------|
| `/` | Go to root level |
| `..` | Go up one level |
| `Tab` | Auto-complete command or show options |
| `?` | Show help for current context |
| `↑` / `↓` | Command history |

### Try It

```
/
```
This takes you to the root. Now try:

```
/interface print
```
This shows all interfaces. Now try:

```
/interface
```
Then press `Tab` — it shows available sub-commands.

Press `Ctrl+C` to clear the current line and return to the prompt. Now  try:

```
/ip address print
```
Shows IP addresses. Press `tab` at any point to see what's available.

### Safe Mode (Critical for Remote Management)

When making changes remotely, a mistake can lock you out. Safe mode protects against this:

1. Press `Ctrl+X` to enter safe mode (you'll see `<SAFE>` in the prompt)
2. Make your changes
3. Press `Ctrl+X` again to exit safe mode and commit changes
4. **If you disconnect while in safe mode, changes are automatically rolled back**

This is a lifesaver when modifying firewall rules or IP addresses remotely.

---

## Lab 3.5 — RoMON

RoMON (Router Management Overlay Network) creates a management network that spans across connected MikroTik devices. Once configured, you can manage any MikroTik device in the chain through any other device — even if you're not directly connected to it.

We configure it now; it becomes useful in Lab 16 when we add the mAP access point.

> **Note:** RoMON requires "advanced" device mode (Lab 3.3). If you skipped that lab, RoMON will show "inactivated, not allowed by device-mode" and won't work.

### Enable RoMON

1. Navigate to **Tools** → **RoMON**

2. Configure:
   - **Enabled:** Checked
   - **Secrets:** Enter a secure password (this will be shared across all your MikroTik devices)

3. Click **Apply** 

4. On the right side under **Configuration**, click **Ports**

5. Double-click the first entry in the list and configure:
   - **Enabled:** Checked
   - **Interface:** all
   - **Secrets:** Same password from step 2

6. Click **Apply** & **OK**

7. Close the RoMON windows.

> **Note:** You won't see any RoMON neighbors yet — there's only one device. When we add the mAP in Lab 16, you'll be able to discover and manage it through RoMON without needing a direct connection.

### Using RoMON (Preview)

Once you have multiple devices:

1. In WinBox, click **Connect to RoMON** (next to the Connect button)
2. After connecting via RoMON, the Neighbors tab shows RoMON-enabled devices
3. Click any device to open a management session through the RoMON tunnel

This is especially useful when your phone is connected to the mAP's Wi-Fi — you can manage both the mAP and the main router from one connection.

---

## Lab 3.6 — Mobile App

MikroTik offers a mobile app for iOS and Android that provides:
- Basic monitoring and configuration
- File transfer (useful for certificate installation in Lab 11)
- RoMON support

### Install the App

1. Search "MikroTik" in your app store

2. Install the application.

### Connect to Your Router

The app works best when your phone is connected to a network managed by your MikroTik. We'll use this in later labs, particularly:
- **Lab 11** — Transferring certificates for EAP authentication
- **Lab 16** — Managing devices when connected to the mAP's Wi-Fi via RoMON

For now, just install the app so it's ready.

---

## Lab 3.9 — RoMON Troubleshooting

*Reference: Use when RoMON discovery fails between devices*

If RoMON-enabled devices aren't discovering each other, the issue is usually mismatched secrets. This troubleshooting method uses RoMON's secret ordering feature.

### The Troubleshooting Process

1. Navigate to **Tools** → **RoMON**

2. Click the **plus button** next to the existing secret to add a blank second entry

3. Click **Apply**

4. Under **Configuration** → **Ports**, double-click the interface entry

5. Add a blank secret entry the same way

6. Click **Apply** & **OK**

7. Repeat on other MikroTik devices

8. Retry discovery

9. If devices now appear, the shared secret was misconfigured. Correct the secrets and remove the blank entries.

### Why This Works

RoMON processes secrets in order:
- Blank then "mysecret" = send unprotected frames, accept protected frames
- "mysecret" then blank = send protected frames, accept unprotected frames
- "mysecret" only = send and accept only protected frames

Adding the blank entry temporarily allows mixed authentication, revealing misconfigured secrets.

---

If you have successfully made it to this point, you can move on to Lab 04.
