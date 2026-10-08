# Appendix C — Additional MikroTik Capabilities

MikroTik devices can do far more than covered in this guide. This appendix provides an overview of additional features with links for further exploration.

### Contents

- [Kid Control](#kid-control) — Parental controls and time-based access
- [The Dude](#the-dude) — Network monitoring and mapping
- [IP Scan](#ip-scan) — Device discovery
- [Netwatch](#netwatch) — Monitor hosts and trigger actions
- [Watchdog](#watchdog) — Auto-reboot on failure
- [Hotspot](#hotspot) — Captive portal (see the Guest Wi-Fi lab)
- [Scripting](#scripting) — Automation and custom logic
- [API Access](#api-access) — Programmatic control

---

## Kid Control

Manage internet access for specific devices — set schedules, time limits, and content filtering.

**Use case:** Home networks with children, or any environment needing time-based access control.

**Documentation:** https://help.mikrotik.com/docs/display/ROS/Kid+Control

**Quick overview:**
- Create profiles with allowed/blocked times
- Assign devices to profiles by MAC address
- Set daily time limits
- Monitor usage

---

## The Dude

Free network monitoring and management software from MikroTik.

**Use case:** Monitor multiple devices, automatic network mapping, alerting.

**Documentation:** https://help.mikrotik.com/docs/display/DUDE/Dude

**Platform limitation:** The Dude client is Windows-only. macOS and Linux users must run it under WINE, and modern macOS (Catalina and later) cannot run it at all due to Apple dropping 32-bit support. The server component runs on RouterOS.

**Key features:**
- Automatic network discovery
- Visual network maps
- SNMP, ICMP, and service monitoring
- Alerts via email, SMS, or sound
- Runs as a package on RouterOS or standalone server

---

## IP Scan

Discover devices on your network.

**Location:** Tools → IP Scan

**Use case:** Find devices, identify MAC addresses, audit network.

**Usage:**
1. Navigate to **Tools** → **IP Scan**
2. Set **Address Range** (e.g., 10.10.255.0/24)
3. Set **Interface** to scan from
4. Click **Start**
5. View discovered devices with IP, MAC, and DNS names

> **On your kit:** With **Interface** `vlan255` and **Address Range** `10.10.255.0/24`, the scan finds your mAP while it is on the 15 cm jumper. With **Interface** `bridge` and `192.168.88.0/24`, it finds your laptop on the backdoor cable.

---

## Netwatch

Monitor IP addresses and trigger actions when they go up or down.

**Location:** Tools → Netwatch

**Use case:** Automated failover, alerts, logging, WAN monitoring.

**Documentation:** https://help.mikrotik.com/docs/display/ROS/Netwatch

### Basic Configuration

1. Navigate to **Tools** → **Netwatch**
2. Click **New**
3. Configure:
   - **Host:** IP address to monitor (e.g., 8.8.8.8)
   - **Interval:** How often to check (e.g., 00:00:30)
   - **Timeout:** How long to wait for response
   - **Up Script:** Commands to run when host becomes reachable
   - **Down Script:** Commands to run when host becomes unreachable

### Example: Log Gateway Status

```
# Down script
:log warning "WAN gateway unreachable!"

# Up script
:log info "WAN gateway restored"
```

### Example: Send Alert via Pushover

```
# Down script (send push notification)
/tool fetch url="https://api.pushover.net/1/messages.json" \
    http-method=post \
    http-data="token=YOUR_APP_TOKEN&user=YOUR_USER_KEY&message=WAN DOWN" \
    keep-result=no
```

The Administrative Cheat Sheet (Appendix F) shows the same idea with Telegram.

### Critical Gotchas (Learned the Hard Way)

These issues were discovered during a real deployment and will save you hours of troubleshooting:

**1. Scripts only fire on state transitions**

Netwatch only runs scripts when the state *changes* from up to down (or vice versa). If a host is already down when Netwatch starts, the down script won't run until the host comes up and then goes down again.

**2. Scripts need special permissions**

Scripts called by Netwatch must have `dont-require-permissions=yes` set, or they won't execute:

```
/system script add name=wan-down-alert dont-require-permissions=yes source={
    :log warning "WAN is down!"
}
```

The Button Script lab hit the same problem. Without **Don't Require Permissions**, its script failed with `not enough permissions`.

**3. Don't monitor local IPs**

If you monitor an IP that exists on the router itself (even on a disabled interface), Netwatch will ping itself and always show "up." This happened when monitoring 192.168.88.1 while a disabled bridge still had that IP assigned.

**4. Some networks block ICMP**

If the upstream gateway blocks ping (ICMP), Netwatch will always show "down" even though the connection works. Options:
- Monitor a different host that responds to ping (8.8.8.8)
- Accept that the monitor will show false "down" status
- Use a different monitoring method (HTTP fetch to a known URL)

**5. Add delay to up scripts**

When a connection recovers, routes need a moment to converge. If your up script tries to send an alert immediately, the fetch may fail. Add a delay:

```
# Up script with delay
:delay 10s
/tool fetch url="https://api.pushover.net/1/messages.json" ...
```

---

## Watchdog

Automatically reboot the device if it becomes unresponsive.

**Location:** System → Watchdog

**Use case:** Remote devices that may hang and need automatic recovery.

**Documentation:** https://help.mikrotik.com/docs/display/ROS/Watchdog

### Two Types of Watchdog

**1. Hardware Watchdog (Watchdog Timer)**

The RouterBOARD has a hardware watchdog that reboots the device if RouterOS stops responding. This catches kernel panics and complete system hangs.

- **Location:** System → Watchdog
- **Setting:** Watchdog Timer = enabled
- This is usually enabled by default

**2. Software Watchdog (Watch Address)**

Monitors an IP address and reboots if it becomes unreachable. This catches network-level issues where the router is running but can't reach the internet.

### Configuring Software Watchdog

1. Navigate to **System** → **Watchdog**

2. Configure:
   - **Watch Address:** IP to ping (e.g., 8.8.8.8)
   - **Ping Start After Boot:** Time to wait after boot before starting checks (e.g., 5m)
   - **Ping Timeout:** How long to wait for response (e.g., 60s)

3. Click **OK**

### When to Use Watchdog

**Good use cases:**
- Remote/unattended devices where physical access is difficult
- Devices with known stability issues
- Temporary deployments where reliability matters more than debugging

**Bad use cases:**
- Production routers where you need to diagnose issues
- Devices where the upstream connection is unstable (will reboot constantly)
- Any situation where rebooting could make things worse

### Important Considerations

- **Ping Start After Boot:** Set this long enough for your WAN connection to establish. If set too short, the router may reboot in a loop because it checks before the connection is up.

- **Choose your watch address carefully:** Don't monitor something that might legitimately be down. 8.8.8.8 (Google DNS) is a common choice because it's highly reliable.

- **Watchdog is a band-aid, not a fix:** If your router needs watchdog to stay running, something is wrong. Use it for reliability while you diagnose the root cause.

> **Reality check:** I've never personally used Watchdog because I'd rather know *why* something failed than have it silently reboot. But for a remote device at a client site where uptime matters more than diagnostics, it's a valid tool.

---

## Hotspot

Captive portal for guest access with authentication, bandwidth limits, and time limits.

**Location:** IP → Hotspot

**Use case:** Guest Wi-Fi, trade show demos, paid internet access, terms acceptance.

**Features:**
- Customizable login pages
- User accounts with time/data limits
- Voucher system
- Walled garden (allow specific destinations without login)
- Integration with User Manager/RADIUS

**Documentation:** https://help.mikrotik.com/docs/display/ROS/Hotspot

> **See the Guest Wi-Fi lab** for a hotspot that redirects guests to a landing page without requiring authentication. Your kit's hotspot is already set up, and the Production Readiness lab deals with its `admin` user.

---

## Scripting

RouterOS has a full scripting language for automation.

**Location:** System → Scripts, System → Scheduler

**Use case:** Automated backups, dynamic configuration, custom monitoring.

**Documentation:** https://help.mikrotik.com/docs/display/ROS/Scripting

**Note:** The RSC Files lab introduces script files, and the Button Script lab has a working script.

---

## API Access

Programmatic access to RouterOS for integration with external systems.

**Supported methods:**
- REST API (RouterOS v7.1+)
- API (raw socket protocol)
- SSH (for scripted commands)

**Use case:** Integration with monitoring systems, automated provisioning, custom tools.

**Documentation:** https://help.mikrotik.com/docs/display/ROS/REST+API

> **In this class:** The HTTPS service you turn on in the certificates lab also serves the REST API, at `https://<router>/rest`. A user needs a group with the `api` and `rest-api` policies. Disabling the `api` service doesn't affect it. The Production Readiness lab limits who can reach the HTTPS service.

---
