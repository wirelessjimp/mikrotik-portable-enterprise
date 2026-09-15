# Lab 23 — Time and NTP

*Prerequisites: Lab 1 (Initial Configuration)*

Accurate time is essential for logging, certificates, and scheduled tasks. This lab configures the MikroTik as both an NTP client (syncing to internet time servers) and an NTP server (serving time to local devices).

---

## Lab 23.1 — System Clock

Before configuring NTP, verify the basic clock settings. Your device may have already picked up the correct time from the class router's NTP server — if so, this is just a verification step.

1. Navigate to **System** → **Clock**

2. Configure:
   - **Time:** Set approximate current time
   - **Date:** Set current date
   - **Time Zone Autodetect:** ✓ Checked
   - **Time Zone Name:** Verify it's correct (or set manually)

3. Click **Apply** and **OK**

---

## Lab 23.2 — NTP Client

Configure the router to sync time from public NTP servers.

1. Navigate to **System** → **NTP Client**

2. Configure:
   - **Enabled:** ✓ Checked
   - **Mode:** unicast
   - **NTP Servers:** Pick two or three servers close to your region:

| Region | Server | Notes |
|--------|--------|-------|
| Global | time.google.com | Anycast, works everywhere |
| Global | pool.ntp.org | Round-robin, regional auto-select |
| North America | time.nist.gov | US government time service |
| North America | time.apple.com | Apple's NTP service |
| Europe | europe.pool.ntp.org | European NTP pool |
| Europe | ntp.se | Swedish national time service |
| Asia-Pacific | asia.pool.ntp.org | Asian NTP pool |

> **Tip:** Two servers is enough. If you're traveling, `time.google.com` and `pool.ntp.org` work from anywhere.

3. Click **Apply**

4. Watch the **Status** field — it should change to **synchronized**

### View NTP Peers

5. Click **Peers** on the right side

6. View the list of NTP servers and their status:
   - **Stratum:** Lower is better (1 = primary server)
   - **Delay:** Round-trip time to server in milliseconds
   - **Offset:** Difference between local clock and server
   - **Jitter:** Variation in delay over time

7. Click **Cancel** to close the Peers window without making changes

> **Note:** If status doesn't sync, verify internet connectivity and DNS resolution.

---

## Lab 23.3 — NTP Server

Configure the MikroTik to serve time to local devices. This reduces NTP traffic to the internet and provides a local time source.

1. Navigate to **System** → **NTP Server**

2. Configure:
   - **Enabled:** ✓ Checked
   - **Broadcast:** ✓ Checked
   - **Multicast:** Unchecked (unless needed)
   - **Manycast:** Unchecked
   - **VRF:** main
   - **Use Local Clock:** ✓ Checked
   - **Local Clock Stratum:** 5

3. Click **Apply** and **OK**

### Configure Clients to Use MikroTik NTP

Local devices can now use the MikroTik as their NTP server:
- **NTP Server:** 10.10.255.1 (or whatever gateway IP the client sees)

For devices that get their NTP server via DHCP, update your DHCP network settings:

4. Navigate to **IP** → **DHCP Server** → **Networks**

5. Edit each network and set **NTP Servers:** to the MikroTik's IP for that VLAN

6. Click **OK**

---

## Lab 23 Summary

| Component | Function |
|-----------|----------|
| System Clock | Basic time/timezone settings |
| NTP Client | Syncs router time from internet |
| NTP Server | Serves time to local devices |

---
