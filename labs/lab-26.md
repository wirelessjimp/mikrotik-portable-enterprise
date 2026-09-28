# Lab 26 — Hotspot (Captive Portal)

*Prerequisites: Lab 1 (Initial Configuration), Lab 7 (Bridges/VLANs), Lab 8 (DHCP)*

Hotspot creates a captive portal — guests connect to Wi-Fi, their device auto-redirects to a landing page. This lab configures a simple redirect-style hotspot where guests are sent directly to your content without requiring login.

**Use cases:**
- Trade show booth — redirect visitors to product info
- Guest Wi-Fi — show terms of service or welcome page
- Demo environments — direct clients to a specific resource

> **Tested Configuration:** This hotspot configuration is what is used in the classroom scenario for students to get a landing page served by an nginx container.

---

## Lab 26.1 — Hotspot Concepts

### How Hotspot Works

1. Client connects to Wi-Fi (or wired port)
2. Client tries to access any website
3. MikroTik intercepts the request and redirects to the login page
4. After authentication (or redirect), client can access allowed resources

### Walled Garden

The "walled garden" defines what clients can access *before* authenticating:
- IP addresses or subnets
- Specific hostnames
- Specific URLs

For a redirect-only hotspot (no login required), you add all your resources to the walled garden and replace the login page with a simple redirect.

---

## Lab 26.2 — Create a Guest Bridge

> **Note:** If you haven't connected your laptop to your main router, do so now and log into your main router.

If you don't already have a guest network, create one:

1. Navigate to **Bridge**

2. Click **New**:
   - **Name:** br-guest
   - **Comment:** Guest network

3. Click **Apply** and **OK**

4. Navigate to **IP** → **Addresses**

5. Click **New**:
   - **Comment:** Guest network
   - **Address:** 10.10.50.1/24
   - **Interface:** br-guest

6. Click **Apply** and **OK**

7. Create a DHCP server for the guest network (see Lab 8):
   - **Create a pool:** Navigate to **IP** → **Pool**, click **Add New**, define a range for your guest subnet
   - **Build the server:** Navigate to **IP** → **DHCP Server**, click **Add New**, set the **Interface** to **br-guest** and assign the pool you just created
   - **Add the network:** Under the **Networks** tab, add the guest subnet with the gateway and DNS pointing to your br-guest IP

---

## Lab 26.3 — Run Hotspot Setup

MikroTik provides a setup wizard that creates most of the configuration:

1. Navigate to **IP** → **Hotspot**

2. Click **Hotspot Setup** on the right-hand side under Actions

3. Follow the wizard:
   - **Hotspot Interface:** br-guest
   - **Local Address of Network:** 10.10.50.1 (should auto-fill)
   - **Masquerade Network:** yes
   - **Address Pool of Network:** 10.10.50.10-10.10.50.254
   - **Select Certificate:** none
   - **IP Address of SMTP Server:** 0.0.0.0 (skip)
   - **DNS Servers:** 10.10.50.1
   - **DNS Name:** (leave blank or enter a name like `guest.local`)
   - **Name of Local Hotspot User:** admin
   - **Password for the User:** [create a password]

4. Click through to complete the wizard

5. The wizard creates:
   - Hotspot server on br-guest
   - Hotspot profile
   - DHCP pool (if not existing)
   - NAT rules
   - DNS configuration

---

## Lab 26.4 — Create Redirect Login Page

When a guest connects to the hotspot, MikroTik serves a login page. We're going to replace the default login page with a simple redirect that sends guests straight to your landing page — no username or password required.

### Create the Redirect Page

1. On your computer, create a text file using your favorite text editor (BBEdit, Notepad++, or the built-in TextEdit on macOS / Notepad on Windows) named `login.html` with this content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Redirecting...</title>
    <meta http-equiv="refresh" content="0; url=http://172.17.0.4">
</head>
<body>
    <p>Redirecting to welcome page...</p>
</body>
</html>
```

> **Note:** If you did Lab 5.4, `172.17.0.4` is the IP address of your web server from that exercise. If you skipped Lab 5, then replace `172.17.0.4` with your actual landing page IP or URL.

### Upload the Login Page

2. In WinBox, navigate to **Files**

3. Find the **hotspot** folder

4. Select the existing `login.html`, then click **Download** in the right-hand panel to save a backup copy to your computer. Then rename it on the router to `login.html.bak`

5. Click **Upload** and select your new `login.html` — it will upload to the root of the file system

6. Drag the uploaded `login.html` from the root into the **hotspot** folder

### How It Works

- Guest connects to Guest network (Wi-Fi or wired)
- Device auto-detects captive portal
- MikroTik serves your login.html
- The meta refresh immediately redirects to your landing page
- Landing page is in walled garden, so it loads without authentication
- Guest has full access to walled garden resources

---

## Lab 26.5 — Configure Walled Garden

Add destinations that guests can access without logging in.

### Walled Garden IP List (IP-Based Access)

The **Walled Garden IP List** tab allows traffic to specific IP addresses without authentication. Use this for containers, internal servers, and any non-web services.

1. In the **IP** → **Hotspot** window, click the **Walled Garden IP List** tab

2. Click **Add New**:
   - **Action:** accept
   - **Server:** hotspot1
   - **Dst. Address:** [IP of your landing page server]

3. Click **Apply** and **OK**

4. Repeat for additional destinations (containers, internal servers, etc.)

### Example: Allow Access to Containers

If running containers from Lab 5:

```
/ip hotspot walled-garden ip add dst-address=172.17.0.2 action=accept comment="OpenSpeedTest"
/ip hotspot walled-garden ip add dst-address=172.17.0.3 action=accept comment="iperf3"
/ip hotspot walled-garden ip add dst-address=172.17.0.4 action=accept comment="nginx content"
```

### Walled Garden by Hostname (HTTP-Based Access)

The **Walled Garden** tab (not IP List) works at the HTTP level, matching on hostnames and URL patterns. Use this when you want to allow access to an external website by name — for example, allowing guests to reach a GitBook page or a public documentation site before logging in.

1. In the **IP** → **Hotspot** window, click the **Walled Garden** tab

2. Click **Add New**:
   - **Action:** allow
   - **Dst. Host:** `*.gitbook.io` (or your specific hostname)

3. Click **Apply** and **OK**

> **Note:** HTTP-based Walled Garden only catches web traffic. If guests need access to non-web services (DNS, NTP, speedtest), use the Walled Garden IP List instead.

---

## Lab 26.6 — Test the Hotspot

1. Add an unused ethernet port to the **br-guest** bridge (**ether6** is a good candidate), then connect a device (phone or laptop) to that port

> **Note:** If your device has an integrated wireless interface (hAP series), you can also create a guest SSID and attach it to br-guest. On non-wireless devices like the hEX S, use a wired connection for testing.

2. The device should detect a captive portal and open a browser

3. You should be redirected to your landing page

4. Verify you can access all walled garden resources

5. Verify you *cannot* access the internet (unless you added internet to the walled garden)

### Troubleshooting

**No captive portal detected:**
- Some devices are slow to detect — try opening a browser manually to http://example.com
- Check that the DHCP server is providing the correct DNS (10.10.50.1)

**Redirect doesn't work:**
- Verify login.html is in the correct hotspot folder
- Check the meta refresh URL is correct
- Verify the destination is in the walled garden

**Can't access landing page:**
- Verify the IP is in the walled garden IP list
- Check firewall rules aren't blocking guest → container traffic

---

## Lab 26.7 — Optional: Allow Internet Access

If you want guests to have internet access after viewing your landing page, you have two options:

### Option 1: Auto-Login (No Authentication)

Configure the hotspot to auto-authenticate clients:

1. Navigate to **IP** → **Hotspot** → **Server Profiles**

2. Edit your profile

3. Set **Login By:** MAC

4. Click **OK**

Now clients are authenticated by MAC address automatically.

### Option 2: Add Internet to Walled Garden

Simply allow all traffic without authentication:

1. Navigate to **IP** → **Hotspot** → **Walled Garden** → **IP**

2. Click **Add New**:
   - **Action:** accept
   - **Dst. Address:** 0.0.0.0/0

3. Click **OK**

> **Warning:** This bypasses all hotspot authentication. Only use if you just want the redirect experience without restricting access.

---

## Lab 26 Summary

| Component | Purpose |
|-----------|---------|
| Hotspot Server | Intercepts traffic, redirects to login page |
| Walled Garden | Defines allowed destinations before login |
| login.html | Custom redirect page |
| Server Profile | Authentication settings |

**Key insight:** The walled garden does the heavy lifting. Once your destinations are whitelisted, clients can access them freely. The login page just provides the redirect trigger.

---
