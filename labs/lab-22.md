# Lab 22 — DNS

*Prerequisites: Lab 1 (your L009). Lab 4 for the nginx container in 22.2.*

DNS is one of those things that "just works" until it doesn't. This lab covers practical DNS configuration on MikroTik — static entries for local hosts, secure upstream DNS, and forcing all DNS through your router.

---

## Lab 22.1 — Understanding MikroTik DNS

By default, your MikroTik acts as a DNS server for clients and forwards queries to upstream servers (typically your ISP's DNS, learned via DHCP).

**Current DNS settings:**

1. Navigate to **IP** → **DNS**

2. Review:
   - **Servers:** Upstream DNS servers (where queries are forwarded)
   - **Dynamic Servers:** DNS servers learned from DHCP client
   - **Allow Remote Requests:** Must be checked for clients to use MikroTik as DNS
   - **Cache Size:** How many entries to cache locally
   - **Cache Used:** Current cache utilization

> **Your kit:** The instructor's script already turned **Allow Remote Requests** on, so your laptop and your containers use the L009 as their DNS server. The servers it forwards to come from the class router.

---

## Lab 22.2 — Static DNS Entries

Create local DNS names for devices on your network. Instead of remembering `172.17.0.4`, you can use `files.lab`.

### See what the script already added

1. Click **IP**, then **DNS**, then **Static**. Two entries are already there, which the instructor's script added: `router.lan` for `192.168.88.1`, and `speedtest.lab` for the OpenSpeedTest container at `172.17.0.2`.
2. In a terminal on your laptop, run `ping speedtest.lab`. The name resolves to `172.17.0.2`, because your laptop asks the L009 for DNS.

### Add your own

3. In the **Static** window, click **New**. Set:
   - **Comment:** nginx content
   - **Name:** `files.lab`
   - **Address:** `172.17.0.4`
   - **TTL:** leave the default

   Click **Apply**, then **OK**.
4. In a terminal on your laptop, run `ping files.lab`. It answers from `172.17.0.4`.
5. Open `http://files.lab` in a browser. It shows the page from your nginx container.

> **Tip:** Use a consistent naming scheme. `.lab`, `.home`, or `.local` are common choices. Avoid `.local` if you have Apple devices, because mDNS conflicts can occur.

---

## Lab 22.3 — Configure Upstream DNS Servers

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

> ### ⚠️ STOP AND READ
> If the network you are on blocks outbound DNS to servers other than its own, your router stops resolving names after this change, and so do your containers. If a name stops resolving, undo it: double-click your WAN DHCP client, check **Use Peer DNS** again, and clear the **Servers** field.

---

## Lab 22.4 — DNS-over-HTTPS (DoH)

Standard DNS queries are unencrypted — your ISP can see every domain you look up. DNS-over-HTTPS encrypts queries to the upstream server.

### Import the Certificate

DoH requires trusting the upstream server's certificate. Cloudflare example:

1. Download the Cloudflare root certificate:
   - Visit https://developers.cloudflare.com/1.1.1.1/encryption/
   - Download the root certificate that page names. This lab was written with the DigiCert Global Root CA. If the page lists a different one now, use that one.

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

## Lab 22.5 — Force All DNS Through MikroTik

Some devices (smart TVs, IoT devices, even Chrome) ignore your DHCP-provided DNS and use hardcoded servers like 8.8.8.8. This bypasses your DNS configuration.

### The Problem

- You configure DNS filtering or DoH
- Smart TV ignores it and queries Google directly
- Your filtering is bypassed

### The Solution: Redirect Port 53

Force all DNS traffic through your router, regardless of what the client requests:

```
/ip firewall nat add chain=dstnat protocol=udp dst-port=53 in-interface-list=LAN action=redirect to-ports=53 comment="Force DNS to router"
/ip firewall nat add chain=dstnat protocol=tcp dst-port=53 in-interface-list=LAN action=redirect to-ports=53 comment="Force DNS to router (TCP)"
```

### Via WinBox

1. Navigate to **IP** → **Firewall** → **NAT**

2. Click **New**

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

If you have a device that legitimately needs to use its own DNS, such as a lab DNS server:

```
/ip firewall nat add chain=dstnat protocol=udp dst-port=53 src-address=192.168.88.50 action=accept comment="Allow device direct DNS"
```

Replace `192.168.88.50` with that device's address.

Place this rule **before** the redirect rules.

---

## Lab 22.6 — DNS Adblock (RouterOS 7.15+)

RouterOS 7.15 introduced native DNS adblock lists. This blocks ads and trackers at the DNS level without external software.

### ⚠️ Here Be Dragons

Before enabling DNS adblock, understand the tradeoffs:

**Pros:**
- Network-wide ad blocking
- No additional hardware or containers needed
- Works on all devices automatically

**Cons:**
- Some websites break when ad domains are blocked
- Requires ongoing blocklist maintenance
- Troubleshooting "why won't this site work?" becomes harder
- Aggressive lists can block legitimate services

### Basic Setup (If You Want to Try It)

1. Navigate to **IP** → **DNS**

2. Increase **Cache Size** to at least 16384 (more if you have RAM)

3. Click **Apply**

4. Navigate to **IP** → **DNS** → **Adlist**

5. Click **New**:
   - **URL:** `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts`
   - **SSL Verify:** yes

6. Click **OK**

7. The list downloads and populates the DNS cache with blocked entries

> **Note:** If nothing downloads, look at the log: `/log/print where message~"adlist"`. A failed certificate check is the likely cause, because the router has no trusted root certificates unless you import them.

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

## Lab 22 Summary

| Feature | Purpose |
|---------|---------|
| Static DNS | Local hostname resolution |
| Upstream servers | Replace ISP DNS |
| DNS-over-HTTPS | Encrypted DNS queries |
| Force DNS redirect | Prevent DNS bypass |
| DNS Adblock | Block ads/trackers (advanced) |

**Recommendation for most users:** Configure static entries for your local devices, use Cloudflare or Quad9 as upstream, enable DoH, and force DNS through the router. Skip the adblock lists unless you're prepared to troubleshoot.
