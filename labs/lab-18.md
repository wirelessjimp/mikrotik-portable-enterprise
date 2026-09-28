# Lab 18 — Scripting & RSC Files

*Prerequisites: Lab 13 (mAP configured with fallback and trunk)*

In Lab 13, you manually built a fallback configuration for the mAP — a bridge, IP address, DHCP server, and Wi-Fi. It took about 20 clicks and commands. Now imagine doing that on 10 devices, or rebuilding it after a factory reset.

This lab teaches you how to turn that manual work into a reusable script. Once you understand RSC files, you can configure a fresh MikroTik device in seconds instead of minutes.

>**Note:** For this lab, you need to make sure you are connected to your mAP on Ether2 because the configurations on the mAP are going to change multiple times.

---

## Lab 18.1 — What Are RSC Files?

An RSC file is a plain text file containing MikroTik CLI commands. When you export a configuration, MikroTik generates an RSC file. When you import an RSC file, MikroTik runs each command in sequence.

### Export vs Backup

| Type | Extension | Contents | Use Case |
|------|-----------|----------|----------|
| Backup | .backup | Binary, encrypted | Restore exact config to same device |
| Export | .rsc | Plain text commands | Recreate config on any device, edit by hand |

**Backup files** are for disaster recovery — restore everything exactly as it was.

**RSC files** are for automation — build a configuration from scratch, copy to multiple devices, or create a template you can customize.

### Viewing an RSC File

1. On your mAP, open **New Terminal**

2. Run:
   ```
   /export
   ```

3. The current configuration scrolls by as CLI commands.

4. To save it to a file:
   ```
   /export file=lab17-current-config
   ```

5. Navigate to **Files** and download `lab17-current-config.rsc`

6. Open the file in any text editor — you'll see commands like:
   ```
   /interface bridge
   add comment="Standalone fallback bridge" name=br-fallback
   add comment="Management bridge VLAN 255" name=br-mgmt
   
   /interface vlan
   add comment="VLAN 20 from trunk" interface=ether1 name=vlan20 vlan-id=20
   ...
   ```

This is your entire configuration as a script.

---

## Lab 18.2 — Anatomy of an RSC File

RSC files follow a predictable structure. Understanding it helps you write your own scripts.

### Command Structure

```
/path/to/menu
command argument1=value1 argument2=value2
```

For example:
```
/ip address
add address=192.168.89.1/27 interface=br-fallback comment="Fallback management IP"
```

### Multiple Commands in Same Menu

When multiple commands target the same menu, you don't repeat the path:

```
/interface vlan
add comment="VLAN 20 from trunk" interface=ether1 name=vlan20 vlan-id=20
add comment="VLAN 30 from trunk" interface=ether1 name=vlan30 vlan-id=30
add comment="VLAN 40 from trunk" interface=ether1 name=vlan40 vlan-id=40
add comment="VLAN 255 from trunk" interface=ether1 name=vlan255 vlan-id=255
```

### Comments in Scripts

Lines starting with `#` are comments:

```
# This section builds the fallback configuration
# for standalone operation without main router

/interface bridge
add name=br-fallback comment="Standalone fallback bridge"
```

---

## Lab 18.3 — The Fallback Configuration as a Script

Here's everything you built in Lab 13.3, converted to a script:

```
# ============================================
# mAP Fallback Configuration Script
# ============================================
# Purpose: Creates standalone access on 192.168.89.0/27
# when mAP is not connected to main router
#
# Use: Import after factory reset or on new device
# ============================================

# Create fallback bridge
/interface bridge
add name=br-fallback comment="Standalone fallback bridge"

# Add ETH2 to fallback bridge
/interface bridge port
add bridge=br-fallback interface=ether2 comment="Fallback ETH2"

# Configure fallback IP
/ip address
add address=192.168.89.1/27 interface=br-fallback comment="Fallback management IP"

# Create DHCP pool
/ip pool
add name=fallback-pool ranges=192.168.89.10-192.168.89.30 comment="Fallback DHCP pool"

# Create DHCP server
/ip dhcp-server
add name=fallback-dhcp interface=br-fallback address-pool=fallback-pool lease-time=01:00:00 add-arp=yes disabled=no

# Create DHCP network
/ip dhcp-server network
add address=192.168.89.0/27 gateway=192.168.89.1 dns-server=192.168.89.1 comment="Fallback network"

# Configure Wi-Fi security
/interface wireless security-profiles
add name=fallback-security mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key="CHANGE_THIS_PASSWORD"

# Configure Wi-Fi interface
/interface wireless
set [ find name=wlan1 ] mode=ap-bridge band=2ghz-onlyn channel-width=20mhz ssid="mAP-Fallback" security-profile=fallback-security country="united states" disabled=no

# Add Wi-Fi to fallback bridge
/interface bridge port
add bridge=br-fallback interface=wlan1 comment="Fallback Wi-Fi"

# ============================================
# End of Fallback Configuration
# ============================================
```

> **Note:** Before using this script, change `CHANGE_THIS_PASSWORD` to your actual Wi-Fi password, and update the country code if needed.

---

## Lab 18.4 — Create Your Own Fallback Script

Let's extract just the fallback portion from your mAP's configuration and save it as a reusable script.

### Build the Fallback Script

1. Open the `lab17-current-config.rsc` file you downloaded in Lab 18.1

2. Save a copy as `mAP-fallback-config.rsc`

3. Delete every section that is NOT related to the fallback configuration. Keep:
   - `/interface bridge` — only `br-fallback`
   - `/interface wireless security-profiles` — only `fallback-psk`
   - `/interface wireless` — only wlan1 with fallback settings
   - `/interface bridge port` — only ether2 and wlan1 in `br-fallback`
   - `/ip pool` — only `fallback-pool`
   - `/ip dhcp-server` — only `fallback-dhcp`
   - `/ip dhcp-server network` — only `192.168.89.0/27`
   - `/ip address` — only `192.168.89.1` on `br-fallback`

4. Add the WPA2 password back into the security profile — exports strip passwords for security

   **Before (from export — password missing):**
   
```
add authentication-types=wpa2-psk mode=dynamic-keys name=fallback-psk
supplicant-identity=""
```

   **After (password added):**

```
add authentication-types=wpa2-psk mode=dynamic-keys name=fallback-psk
wpa2-pre-shared-key="YourFallbackPassword" supplicant-identity=""
```

   > **Watch out:** `supplicant-identity` is NOT the password — it's a client identity field used in 802.1X. The password goes in `wpa2-pre-shared-key`. This is the most common mistake when editing exported security profiles.

> **Why this approach?** MikroTik exports are in dependency order — pools before servers, bridges before ports. Deleting lines preserves that order. Building from scratch requires you to get the order right yourself.

5. Add comments explaining each section. Use `#` at the beginning of a line for comments:
   ```
   # Fallback bridge for emergency access
   /interface bridge add name=br-fallback
   
   # Fallback IP address
   /ip address add address=192.168.89.1/27 interface=br-fallback
   ```

6. Save your changes

You now have a reusable script for the fallback configuration.

> **Checkpoint:** Your finished script should be around 25-30 lines (without comments). If it's significantly longer, you're including sections that aren't fallback. If it's under 15 lines, you're missing something — check the list in step 3.

---

## Lab 18.5 — Loading Scripts onto a Fresh Device

Now let's test the script on a fresh device (or the same device after a reset).

### Reset and Restore (Class Exercise)

This is the real test — prove your script works by destroying the config and rebuilding from it.

1. Navigate to **System** → **Reset Configuration**
   - Check **No Default Configuration**
   - Click **Reset Configuration**
   - Click **OK**

2. The mAP reboots with a blank config. Ensure your laptop is directly connected to **ether2** on the mAP

3. Open WinBox and connect via MAC discovery (no IP address or password exists yet)

4. Navigate to **Files** and upload your `mAP-fallback-config.rsc` from your laptop

5. Open **New Terminal** and run:

```
/import file-name=mAP-fallback-config.rsc
```

7. Watch each command execute. When complete, WinBox will likely disconnect — the network configuration just changed underneath you

> **Alternative:** If `/import` fails, open the script in a text editor on your laptop, select all, copy, and paste directly into the WinBox terminal. This bypasses the import command and executes each line individually.

8. Verify: disconnect from ether2, connect to the **mAP-Fallback** SSID, and confirm you get a 192.168.89.x address

> **If it fails:** This is the learning moment. Read the error, find the missing or broken line in your script, fix it, and try again. Your classmates' scripts may have different errors — help each other debug.

### Restore Full Configuration

The fallback script proved your scripting skills, but the mAP needs its full configuration back to continue with the remaining labs.

1. Upload the binary backup you created at the end of Lab 13 (`mAP-backup.backup`) to the mAP via **Files**

2. Select your backup file and click **Restore**

3. The mAP reboots with the complete configuration — VLANs, bridges, WireGuard, everything

> **This is why we back up.** The RSC script rebuilt one piece. The binary backup restores everything. Different tools, different jobs — just like Lab 6 explained.

---

## Lab 18.6 — Building Modular Scripts

As you build more configurations, you'll want to organize scripts by function. Here's a recommended structure:

### Modular Approach

Instead of one giant script, create multiple scripts for different purposes:

| Script | Purpose |
|--------|---------|
| `base-config.rsc` | Identity, password, timezone, NTP |
| `fallback-config.rsc` | Standalone fallback access |
| `trunk-config.rsc` | VLAN interfaces for trunk connection |
| `wireguard-config.rsc` | WireGuard tunnel setup |
| `romon-config.rsc` | RoMON configuration |
| `wifi-mgmt-config.rsc` | Management Wi-Fi SSID |

Then, to configure a new mAP:
```
/import file-name=base-config.rsc
/import file-name=fallback-config.rsc
/import file-name=trunk-config.rsc
/import file-name=wireguard-config.rsc
/import file-name=romon-config.rsc
/import file-name=wifi-mgmt-config.rsc
```

### Using Variables

RouterOS supports variables in scripts. This is useful for device-specific values:

```
# Set device-specific values
:local deviceName "mAP-Remote"
:local mgmtIP "10.10.255.2/24"
:local wgPublicKey "abc123..."

# Use variables in commands
/system identity set name=$deviceName
/ip address add address=$mgmtIP interface=br-mgmt
```

> **Note:** Variable syntax is more advanced. For most use cases, simple find-and-replace in a text editor is sufficient.

---

## Lab 18.7 — Trunk Configuration Script

Here's the trunk configuration from Lab 13.4 as a script:

```
# ============================================
# mAP Trunk Configuration Script
# ============================================
# Purpose: Configure VLAN interfaces and management
# for connection to main router
#
# Prerequisites: Base config must be applied first
# ============================================

# Create VLAN interfaces on ether1 (trunk port)
/interface vlan
add name=vlan20 vlan-id=20 interface=ether1 comment="VLAN 20 from trunk"
add name=vlan30 vlan-id=30 interface=ether1 comment="VLAN 30 from trunk"
add name=vlan40 vlan-id=40 interface=ether1 comment="VLAN 40 from trunk"
add name=vlan255 vlan-id=255 interface=ether1 comment="VLAN 255 from trunk"

# Create management bridge
/interface bridge
add name=br-mgmt comment="Management bridge VLAN 255"

# Add VLAN 255 to management bridge
/interface bridge port
add bridge=br-mgmt interface=vlan255 comment="VLAN 255 to management bridge"

# Configure management IP
/ip address
add address=10.10.255.2/24 interface=br-mgmt comment="mAP management IP"

# Configure default route via main router
/ip route
add dst-address=0.0.0.0/0 gateway=10.10.255.1 comment="Default route via main router"

# Configure DNS
/ip dns
set servers=10.10.255.1

# ============================================
# End of Trunk Configuration
# ============================================
```

---

## Lab 18.8 — WireGuard Configuration Script

Here's the WireGuard configuration from Lab 13.5 as a script:

```
# ============================================
# mAP WireGuard Configuration Script
# ============================================
# Purpose: Configure WireGuard tunnel to main router
#
# IMPORTANT: Update the following before running:
# - MAIN_ROUTER_PUBLIC_KEY: Your main router's WG public key
# - DDNS_ADDRESS: Your DDNS hostname from Lab 14.4
# ============================================

# Create WireGuard interface
/interface wireguard
add name=wg-home listen-port=51820 mtu=1420

# Configure WireGuard IP
/ip address
add address=10.255.255.2/24 interface=wg-home comment="WireGuard tunnel IP"

# Add peer (main router)
# UPDATE THESE VALUES:
/interface wireguard peers
add interface=wg-home \
    public-key="MAIN_ROUTER_PUBLIC_KEY" \
    endpoint-address="DDNS_ADDRESS" \
    endpoint-port=51820 \
    allowed-address=10.255.255.0/24,10.10.0.0/16 \
    persistent-keepalive=25s \
    comment="Main router"

# ============================================
# After running this script:
# 1. Get this device's public key: /interface wireguard print
# 2. Add this device as a peer on the main router
# ============================================
```

> **Note:** WireGuard generates a new key pair each time you create an interface. You'll need to get the public key after running this script and add it to your main router.

---

## Lab 18.9 — RoMON Configuration Script

```
# ============================================
# mAP RoMON Configuration Script
# ============================================
# Purpose: Enable RoMON for remote device management
#
# IMPORTANT: Update ROMON_SECRET before running
# Use the same secret on all RoMON devices
# ============================================

# Enable RoMON
/tool romon
set enabled=yes secrets="ROMON_SECRET"

# Add RoMON ports
/tool romon port
add interface=ether1 forbid=no cost=100
add interface=br-fallback forbid=no cost=100
add interface=br-mgmt forbid=no cost=100

# ============================================
# End of RoMON Configuration
# ============================================
```

---

## Lab 18.10 — Complete mAP Deployment Script

Here's everything combined into a single deployment script. This configures a factory-fresh mAP with all the features from Lab 13:

```
# ============================================
# Complete mAP Deployment Script
# ============================================
# Purpose: Full mAP configuration from factory reset
# Version: 1.0
# 
# BEFORE RUNNING - UPDATE THESE VALUES:
# - DEVICE_NAME: Identity for this device
# - ADMIN_PASSWORD: Admin password
# - FALLBACK_WIFI_PASSWORD: Fallback Wi-Fi PSK
# - MGMT_WIFI_PASSWORD: Management Wi-Fi PSK
# - MAIN_ROUTER_WG_PUBKEY: Main router's WireGuard public key
# - DDNS_ADDRESS: Your DDNS hostname
# - ROMON_SECRET: Shared RoMON secret
# - COUNTRY_CODE: Your country (e.g., "united states")
# ============================================

#
# BASE CONFIGURATION
#
/system identity
set name=DEVICE_NAME

/user
set [find name=admin] password=ADMIN_PASSWORD

#
# FALLBACK CONFIGURATION (192.168.89.0/27)
#
/interface bridge
add name=br-fallback comment="Standalone fallback bridge"

/interface bridge port
add bridge=br-fallback interface=ether2 comment="Fallback ETH2"

/ip address
add address=192.168.89.1/27 interface=br-fallback comment="Fallback management IP"

/ip pool
add name=fallback-pool ranges=192.168.89.10-192.168.89.30

/ip dhcp-server
add name=fallback-dhcp interface=br-fallback address-pool=fallback-pool lease-time=01:00:00 add-arp=yes

/ip dhcp-server network
add address=192.168.89.0/27 gateway=192.168.89.1 dns-server=192.168.89.1

/interface wireless security-profiles
add name=fallback-security mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key="FALLBACK_WIFI_PASSWORD"

/interface wireless
set [find name=wlan1] mode=ap-bridge band=2ghz-onlyn channel-width=20mhz ssid="mAP-Fallback" security-profile=fallback-security country="COUNTRY_CODE" disabled=no

/interface bridge port
add bridge=br-fallback interface=wlan1 comment="Fallback Wi-Fi"

#
# TRUNK CONFIGURATION
#
/interface vlan
add name=vlan20 vlan-id=20 interface=ether1 comment="VLAN 20"
add name=vlan30 vlan-id=30 interface=ether1 comment="VLAN 30"
add name=vlan40 vlan-id=40 interface=ether1 comment="VLAN 40"
add name=vlan255 vlan-id=255 interface=ether1 comment="VLAN 255"

/interface bridge
add name=br-mgmt comment="Management bridge"

/interface bridge port
add bridge=br-mgmt interface=vlan255

/ip address
add address=10.10.255.2/24 interface=br-mgmt comment="Management IP"

/ip route
add dst-address=0.0.0.0/0 gateway=10.10.255.1

/ip dns
set servers=10.10.255.1

#
# WIREGUARD CONFIGURATION
#
/interface wireguard
add name=wg-home listen-port=51820 mtu=1420

/ip address
add address=10.255.255.2/24 interface=wg-home comment="WireGuard IP"

/interface wireguard peers
add interface=wg-home public-key="MAIN_ROUTER_WG_PUBKEY" endpoint-address="DDNS_ADDRESS" endpoint-port=51820 allowed-address=10.255.255.0/24,10.10.0.0/16 persistent-keepalive=25s comment="Main router"

#
# ROMON CONFIGURATION
#
/tool romon
set enabled=yes secrets="ROMON_SECRET"

/tool romon port
add interface=ether1 forbid=no cost=100
add interface=br-fallback forbid=no cost=100
add interface=br-mgmt forbid=no cost=100

#
# MANAGEMENT WI-FI
#
/interface wireless security-profiles
add name=mgmt-security mode=dynamic-keys authentication-types=wpa2-psk wpa2-pre-shared-key="MGMT_WIFI_PASSWORD"

/interface wireless
add name=wlan2 master-interface=wlan1 mode=ap-bridge ssid="LabMgmt" security-profile=mgmt-security

/interface bridge port
add bridge=br-mgmt interface=wlan2 comment="Management Wi-Fi"

# ============================================
# DEPLOYMENT COMPLETE
#
# Next steps:
# 1. Get this device's WireGuard public key:
#    /interface wireguard print
# 2. Add as peer on main router
# 3. Test connectivity
# ============================================
```

---

## Lab 18 — Script Reference

The following sections provide ready-made scripts for common configurations. These are not hands-on exercises — use them as templates when building your own deployments.

### Method 1: Upload and Import via WinBox

1. Connect to the target device via WinBox

2. Navigate to **Files**

3. Use the Upload button to upload `.rsc` file into the Files window (or use the Upload button)

4. Open **New Terminal**

5. Run:
   ```
   /import file-name=mAP-fallback-config.rsc
   ```

6. Watch the terminal — each command executes and shows its result

7. If there are errors, the terminal shows which line failed

### Method 2: Copy/Paste into Terminal

For quick testing or small scripts:

1. Connect to the device via WinBox

2. Open **New Terminal**

3. Open your `.rsc` file in a text editor

4. Copy the entire contents

5. Right-click in the WinBox terminal and paste

6. Commands execute immediately

> **Warning:** Be careful with copy/paste on large scripts. If the connection drops mid-paste, you'll have a partial configuration.

### Method 3: FTP/SFTP Upload

For automated deployment:

1. Enable FTP or SSH on the target device

2. Upload the `.rsc` file via FTP/SFTP to the device's file system

3. SSH in and run:
   ```
   /import file-name=mAP-fallback-config.rsc
   ```

---

## Lab 18 Summary

You now understand:

- ✅ The difference between .backup (binary) and .rsc (text) files
- ✅ How RSC files are structured
- ✅ How to export your configuration as a script
- ✅ How to create modular, reusable scripts
- ✅ How to deploy a complete configuration from script

**The payoff:** You can now configure a factory-fresh MikroTik device in under a minute by importing a script. No more clicking through 50 menus.

---
