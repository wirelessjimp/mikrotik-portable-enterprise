# Lab 15 — MikroTik Cloud

*Prerequisites: Lab 4 (WAN access configured)*

MikroTik Cloud provides a small set of free cloud-based services for your router. You've already used the most important one — DDNS — in Lab 12. This lab covers what else is available and how to verify everything is configured correctly.

---

## Lab 15.1 — Cloud Tab Overview

Navigate to **IP** → **Cloud** and click the **Cloud** tab (the leftmost tab).

You'll see the following fields:

| Field | Description |
|-------|-------------|
| **DDNS Enabled** | Set to `auto` (enabled automatically when BTH is active) or `yes` (always on) |
| **DDNS Update Interval** | How often the router checks for IP changes. `01:00:00` is fine for most use cases |
| **Update Time** | When checked, the router syncs its clock from MikroTik's cloud servers on reboot and IP change |
| **Public Address** | Your router's current public IPv4 address as seen by the cloud server |
| **Public IPv6 Address** | Your router's public IPv6 address (if applicable) |
| **DNS Name** | Your DDNS hostname (e.g., `abc123def456.sn.mynetname.net`) |

> **Update Time:** If you don't have an NTP server configured, leave **Update Time** checked. It keeps the router clock accurate using MikroTik's cloud servers, which matters for certificate validity, HTTPS, and log timestamps.

### Verify Your Configuration

Confirm the following are set correctly:
- **DDNS Enabled:** `auto` or `yes`
- **Update Time:** Checked
- **DNS Name:** Populated with your `.sn.mynetname.net` address
- **Status:** `updated`

If **Status** shows `updating...` wait a moment and check again. If it shows an error, verify your WAN connection is working and click **Force Update** in the Actions column on the right.

---

## Lab 15.2 — Remote Management

MikroTik Cloud does not provide a web portal for remote router management. Remote access to your router is handled by the VPN infrastructure you built in Labs 12-14:

- **Manual WireGuard** (Lab 12-13) — direct tunnel to your router, split tunnel, full Winbox/WebFig access
- **Back to Home** (Lab 14) — relay-based tunnel, works through NAT

Third-party cloud management services for MikroTik exist, but they are outside the scope of this guide.

---

## Lab 15.3 — Cloud Backup

MikroTik Cloud provides one free encrypted backup slot per device (CLI only, 15MB max). For the hardware in this guide, the local backup workflow from Lab 6 is simpler and more practical.

If you want to explore cloud backup, see the MikroTik documentation at:
`https://help.mikrotik.com/docs/spaces/ROS/pages/97779929/Cloud`

For day-to-day use, stick with Lab 6 local backups.

---

## Lab 15 Summary

You now have:
- ✅ DDNS active and updating
- ✅ Clock synchronization via Update Time
- ✅ Remote access via WireGuard (Labs 12-14)

The Cloud tab is intentionally simple — MikroTik keeps these services lightweight and free. The real value was already captured when you set up DDNS and BTH VPN.

> **Email Notifications:** MikroTik supports sending email alerts via scripting and Netwatch. This is covered in a later lab alongside other automation and alerting topics.

---

# Lab Notes — Labs 11-15

Print this page or copy to a document for recording important values.

---

**Lab 11 — User Manager**

| Item | Value |
|------|-------|
| CA Certificate Name | radius-ca |
| Server Certificate Name | radius-server |
| EAP-TLS User | user1@mikrotik.test |
| EAP-PEAP User | user2@mikrotik.test |
| EAP-PEAP Password | |
| RADIUS Shared Secret | |
| Enterprise AP IP | |

---

**Labs 12-13 — WireGuard Server**

| Item | Value |
|------|-------|
| Server Interface Name | wg-server |
| Server Listen Port | 51820 |
| Server Public Key | |
| VPN Subnet | 10.255.255.0/24 |
| DDNS Address | `.sn.mynetname.net` |

**Peers**

| Peer Name | Public Key | VPN IP | Notes |
|-----------|------------|--------|-------|
| | | | |
| | | | |
| | | | |

---

**Lab 14 — Back to Home VPN**

| Item | Value |
|------|-------|
| BTH VPN Status | |
| VPN DNS Name | `.vpn.mynetname.net` |
| VPN Port | |

---

**Lab 15 — Cloud**

| Item | Value |
|------|-------|
| DDNS Enabled | auto / yes |
| DNS Name | `.sn.mynetname.net` |
| Public Address | |
| Update Time | Checked |

---

*Document Version: 3.0*
*Last Updated: May 2026*
