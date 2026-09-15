# Lab 27 — DNS Configuration

*Prerequisites: Lab 1 (Initial Configuration), Lab 8 (DHCP)*

DNS is one of those things that "just works" until it doesn't. This lab covers practical DNS configuration on MikroTik — static entries for local hosts, secure upstream DNS, and forcing all DNS through your router.

---

## Lab 27.1 — Understanding MikroTik DNS

By default, your MikroTik acts as a DNS server for clients and forwards queries to upstream servers (typically your ISP's DNS, learned via DHCP).

**Current DNS settings:**

1. Navigate to **IP** → **DNS**

2. Review:
   - **Servers:** Upstream DNS servers (where queries are forwarded)
   - **Dynamic Servers:** DNS servers learned from DHCP client
   - **Allow Remote Requests:** Must be checked for clients to use MikroTik as DNS
   - **Cache Size:** How many entries to cache locally
   - **Cache Used:** Current cache utilization

---

## Lab 27.2 — Static DNS Entries

Create local DNS names for devices on your network. Instead of remembering 10.10.255.50, you can use `server.lab` or `nas.home`.

### Add a Static Entry

1. Navigate to **IP** → **DNS** → **Static**

2. Click **Add New**:
   - **Name:** server.lab
   - **Address:** 10.10.255.50
   - **TTL:** 1d (or leave default)
   - **Comment:** Lab server

3. Click **OK**

### Common Static Entries

```
/ip dns static add name=router.lab address=10.10.255.1 comment="Main router"
/ip dns static add name=nas.lab address=10.10.255.10 comment="NAS"
/ip dns static add name=speedtest.lab address=172.17.0.2 comment="OpenSpeedTest container"
/ip dns static add name=files.lab address=172.17.0.4 comment="nginx content server"
```

### Test the Entry

4. From a client on your network, ping the hostname:
   ```
   ping server.lab
   ```

5. The name should resolve to the IP you configured

> **Tip:** Use a consistent naming scheme. `.lab`, `.home`, or `.local` are common choices. Avoid `.local` if you have Apple devices — mDNS conflicts can occur.

---

## Lab 27.3 — Configure Upstream DNS Servers

Replace your ISP's DNS with something faster, more private, or more reliable.

### Popular Public DNS Options

| Provider | Primary | Secondary | Notes |
|----------|---------|-----------|-------|
| Cloudflare | 1.1.1.1 | 1.0.0.1 | Fast, privacy-focused |
| Google | 8.8.8.8 | 8.8.4.4 | Reliable, logs queries |
| Quad9 | 9.9.9.9 | 149.112.112.112 | Security-focused, blocks malware |
| OpenDNS | 208.67.222.222 | 208.67.220.220 | Filtering options available |

### Set Static DNS Servers

1. Navigate to **IP** → **DNS**

2. In **Servers**, enter your preferred DNS:
   ```
   1.1.1.1,1.0.0.1
   ```

3. Click **Apply**

### Remove Dynamic DNS (ISP servers)

If your WAN uses DHCP, the router learns DNS from your ISP. To use only your configured servers:

4. Navigate to **IP** → **DHCP Client**

5. Double-click your WAN DHCP client

6. Uncheck **Use Peer DNS**

7. Click **OK**

8. Return to **IP** → **DNS** and verify Dynamic Servers is now empty

---

## Lab 27.4 — DNS-over-HTTPS (DoH)

Standard DNS queries are unencrypted — your ISP can see every domain you look up. DNS-over-HTTPS encrypts queries to the upstream server.

### Import the Certificate

DoH requires trusting the upstream server's certificate. Cloudflare example:

1. Download the Cloudflare root certificate:
   - Visit https://developers.cloudflare.com/1.1.1.1/encryption/
   - Download the DigiCert Global Root CA certificate

2. Navigate to **Files** in WinBox

3. Upload the certificate file

4. Navigate to **System** → **Certificates**

5. Click **Import**

6. Select the uploaded certificate file

7. Click **Import** again

### Configure DoH

8. Navigate to **IP** → **DNS**

9. In **DoH Server**, enter:
   ```
   https://cloudflare-dns.com/dns-query
   ```

10. Check **Verify DoH Certificate**

11. Click **Apply**

### How DoH Works with Standard DNS

- RouterOS tries DoH first
- If DoH fails, it falls back to standard DNS servers
- Keep standard servers configured as backup

### Verify DoH is Working

12. Navigate to https://1.1.1.1/help in a browser on a client device

13. It should show "Using DNS over HTTPS (DoH): Yes"

> **Note:** Some DoH providers have known issues with MikroTik. Cloudflare is well-tested. Quad9 may show errors in logs but still functions.

---

## Lab 27.5 — Force All DNS Through MikroTik

Some devices (smart TVs, IoT devices, even Chrome) ignore your DHCP-provided DNS and use hardcoded servers like 8.8.8.8. This bypasses your DNS configuration.

### The Problem

- You configure DNS filtering or DoH
- Smart TV ignores it and queries Google directly
- Your filtering is bypassed

### The Solution: Redirect Port 53

Force all DNS traffic through your router, regardless of what the client requests:

```
/ip firewall nat add chain=dstnat protocol=udp dst-port=53 action=redirect to-ports=53 comment="Force DNS to router"
/ip firewall nat add chain=dstnat protocol=tcp dst-port=53 action=redirect to-ports=53 comment="Force DNS to router (TCP)"
```

### Via WinBox

1. Navigate to **IP** → **Firewall** → **NAT**

2. Click **Add New**

3. On the **General** tab:
   - **Chain:** dstnat
   - **Protocol:** udp
   - **Dst. Port:** 53

4. On the **Action** tab:
   - **Action:** redirect
   - **To Ports:** 53

5. Add a **Comment:** "Force DNS to router"

6. Click **OK**

7. Repeat for TCP (some DNS uses TCP)

### Exclude Specific Devices (Optional)

If you have a device that legitimately needs to use its own DNS (like a Pi-hole):

```
/ip firewall nat add chain=dstnat protocol=udp dst-port=53 src-address=10.10.255.100 action=accept comment="Allow Pi-hole direct DNS"
```

Place this rule **before** the redirect rules.

---

## Lab 27.6 — DNS Adblock (RouterOS 7.15+)

RouterOS 7.15 introduced native DNS adblock lists. This blocks ads and trackers at the DNS level without external software.

### ⚠️ Here Be Dragons

Before enabling DNS adblock, understand the tradeoffs:

**Pros:**
- Network-wide ad blocking
- No additional hardware/containers needed
- Works on all devices automatically

**Cons:**
- Some websites break when ad domains are blocked
- Requires ongoing blocklist maintenance
- Troubleshooting "why won't this site work?" becomes harder
- Aggressive lists can block legitimate services

**If you've tried Pi-hole or similar and found it frustrating, this will be similar.**

### Basic Setup (If You Want to Try It)

1. Navigate to **IP** → **DNS**

2. Increase **Cache Size** to at least 16384 (more if you have RAM)

3. Click **Apply**

4. Navigate to **IP** → **DNS** → **Adlist**

5. Click **Add New**:
   - **URL:** `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts`
   - **SSL Verify:** yes

6. Click **OK**

7. The list downloads and populates the DNS cache with blocked entries

### If Things Break

Websites not loading? Forms not submitting? Videos not playing?

1. Navigate to **IP** → **DNS** → **Adlist**

2. Disable the adlist (uncheck **Enabled**)

3. Flush the DNS cache:
   ```
   /ip dns cache flush
   ```

4. Test again

### Alternative: Use a Filtering DNS Provider

For simpler ad blocking without managing lists yourself, point your upstream DNS to a filtering provider:

| Provider | DNS Servers | What It Blocks |
|----------|-------------|----------------|
| Quad9 | 9.9.9.9 | Malware, phishing |
| CleanBrowsing | 185.228.168.9 | Adult content + security |
| AdGuard DNS | 94.140.14.14 | Ads + trackers |
| NextDNS | Custom | Configurable (account required) |

This is "set and forget" with no blocklist management.

---

## Lab 27 Summary

| Feature | Purpose |
|---------|---------|
| Static DNS | Local hostname resolution |
| Upstream servers | Replace ISP DNS |
| DNS-over-HTTPS | Encrypted DNS queries |
| Force DNS redirect | Prevent DNS bypass |
| DNS Adblock | Block ads/trackers (advanced) |

**Recommendation for most users:** Configure static entries for your local devices, use Cloudflare or Quad9 as upstream, enable DoH, and force DNS through the router. Skip the adblock lists unless you're prepared to troubleshoot.

---

# Lab Notes — Lab 27

| Item | Value |
|------|-------|
| Upstream DNS Servers | |
| DoH Server URL | |
| Local Domain Suffix | .lab / .home / other: |

**Static DNS Entries:**

| Hostname | IP Address |
|----------|------------|
| | |
| | |
| | |
| | |

---

*Document Version: Draft 1.0*
*Last Updated: March 2026*
