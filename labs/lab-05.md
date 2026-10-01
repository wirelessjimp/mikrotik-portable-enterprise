# Lab 5 — Containers

*Prerequisites: Lab 1, Lab 2 (container package installed, USB formatted), Lab 3 (device mode set to advanced)*

> **WinBox Tip:** As you work through the labs, you'll open multiple windows (Interfaces, Bridge, IP, etc.). WinBox keeps these open in the background even when you navigate elsewhere. Click the window icon in the top bar (next to Workspace) to see all open windows and switch between them, instead of reopening from the left-hand menu each time.
> 
> ![WinBox windows](images/winbox-tip.png)

Containers allow you to run services directly on the MikroTik router. This lab builds several useful containers: a speedtest server, iperf3 server, and nginx content server.

> **⚠️ HARDWARE COMPATIBILITY WARNING:** Not all MikroTik routers can run standard container images.
>
> **Devices that work with this lab:**
> - L009 (has USB, arm v7)
> - RB5009 (has USB, arm64)
> - hAP ax³ (has USB, arm64)
>
> **Devices that will NOT work:**
> - hEX S Refresh, hEX Refresh (EN7562CT CPU — only supports arm32v5 images, which most containers don't provide)
> - hAP ax² (CPU is fine but no USB port for container storage)
>
> If you attempt containers on an unsupported device, you'll see "Illegal instruction" or "Signal 4" errors. The Architecture Name in System → Resources shows "arm" for both working and non-working ARM devices, so that check won't help you — refer to this list instead.

> **📝 SYNTAX NOTE (RouterOS 7.20+):** Container commands changed in RouterOS 7.20. This guide uses the new syntax. If you're running 7.19.x or earlier, use the old syntax:
>
> | New (7.20+) | Old (7.19 and earlier) |
> |-------------|------------------------|
> | `/container/envs/add list=name` | `/container/envs/add name=name` |
> | `/container/mounts/add list=name` | `/container/mounts/add name=name` |
> | `/container/add ... envlists=name mountlists=name` | `/container/add ... envlist=name mounts=name` |

**Requirements for containers:**
- Architecture: arm (v6+), arm64, or x86 — **not arm32v5**
- RouterOS: v7.4 or newer
- RAM: 512MB minimum (1GB recommended for multiple containers)
- Storage: External USB recommended

> **Resource Considerations:** Running multiple containers consumes RAM. On devices with 512MB (L009, hEX S), limit to 2-3 active containers. After adding each container, we'll check resource usage to understand the impact.

---

## Lab 5.0 — Enable Container Mode

Before building containers, we need to enable container support and explore the environment.

### Enable Container Mode

1. Open a Terminal and run:

   ```
   /system/device-mode/update container=yes
   ```

2. The router prompts you to confirm by pressing the reset button. **Press and hold the reset button firmly** until the port LEDs turn off, then release.

3. The router reboots. Wait for it to come back online and reconnect.

> **Note:** If you already set device-mode to "advanced" in Lab 3.3, container support may already be enabled. Check with `/system/device-mode/print` — if `container: yes` is shown, skip this step.

### Explore Before Building

Let's look at what exists before we create anything:

4. Navigate to **Containers** in the left menu.

   The container list is empty — we haven't created any yet.

5. Navigate to **Files** in the left menu.

   You should see your USB drive (`usb1`) listed. This is where container data will live.

6. Open a Terminal and check current resource usage:

   ```
   /system resource print
   ```

   Note the **free-memory** value. We'll compare this after adding containers.

---

## Lab 5.1 — Container Networking

All containers share a common network infrastructure. We'll create this once, then each container connects to it.

### IP Addressing Scheme

| Item | Address | Notes |
|------|---------|-------|
| Container bridge gateway | 172.17.0.254 | Router's interface to container network |
| Reserved | 172.17.0.1 | Reserved for future use |
| OpenSpeedTest | 172.17.0.2 | veth2 |
| iperf3 | 172.17.0.3 | veth3 |
| nginx | 172.17.0.4 | veth4 |
| Pi-hole (optional) | 172.17.0.5 | veth-pihole (Lab 5.9) |

> **Pattern:** The veth interface number matches the last octet of the IP address. Easy to remember, easy to extend.

### Create the Container Bridge

1. Open a Terminal and create the bridge:

   ```
   /interface/bridge/add name=dockers comment="Container network"
   ```

2. Assign an IP address to the bridge:

   ```
   /ip/address/add address=172.17.0.254/24 interface=dockers comment="Container gateway"
   ```
### Add Container Network to LAN Interface List

3. The container network needs to be in the LAN interface list so containers can reach the router's DNS server and other services.

```
/interface/list/member/add list=LAN interface=dockers comment="Container network"
```

### Configure Container Registry

4. Tell RouterOS where to pull container images from:

   ```
   /container/config/set registry-url=https://registry-1.docker.io tmpdir=/usb1/pull
   ```

   > **Note:** `tmpdir` specifies where images are downloaded before extraction. This must be on the USB drive to avoid filling internal storage.

---

## Lab 5.2 — OpenSpeedTest Container

OpenSpeedTest provides an HTML5-based speed test accessible from any browser. No app needed — just connect and test.

### Create Virtual Interface

```
/interface/veth/add name=veth2 address=172.17.0.2/24 gateway=172.17.0.254 comment="OpenSpeedTest"
```

### Add to Container Bridge

```
/interface/bridge/port/add bridge=dockers interface=veth2 comment="OpenSpeedTest"
```

### Set Environment Variables

```
/container/envs/add list=speedtest_envs key=TZ value="America/Denver"
```

> **Note:** Change the timezone to match your location. Examples: `Europe/London`, `Asia/Tokyo`, `America/New_York`

### Create Mount Point

First, create the mount:

```
/container/mounts/add list=speedtest_mount src=/usb1/speedtest-logs dst=/var/log comment="OpenSpeedTest logs"
```

### Deploy Container

```
/container/add remote-image=openspeedtest/latest interface=veth2 root-dir=/usb1/speedtest mountlists=speedtest_mount envlists=speedtest_envs dns=172.17.0.254 start-on-boot=yes comment="OpenSpeedTest"
```
> **Prerequisite:** The container uses the MikroTik as its DNS server (172.17.0.254). This works because the default configuration has DNS remote requests enabled. You can verify with `/ip/dns/print` — look for `allow-remote-requests: yes`.

### Start the Container

1. Navigate to **Containers** in the left menu

2. The first column shows a **flag** indicating container state, though the column header doesn't label it as such:
   - *(blank)* — Extracting or stopped
   - **R** — Running

   > **Tip:** In the CLI, `/container print` explicitly shows what the flags mean (e.g., `Flags: R - RUNNING`), which can be clearer than the GUI.

3. Wait for the container image to finish extracting (may take a minute or two depending on image size and USB speed).

4. Select the container and click **Start** under Actions on the right.

5. The flag column should show **R** when running. Verify with:

   ```
   /container print
   ```

6. If you see `error` in the comment or the container won't start, check the log:

   ```
   /log print where topics~"container"
   ```

### Test

Open a browser and navigate to: **http://172.17.0.2:3000**

You should see the OpenSpeedTest interface.

> **Performance Note:** The L009 is a capable router but underpowered for running a speedtest server alongside its routing duties. During testing, you may see CPU hit 100% and speeds cap around 400-500 Mbps — this is the L009's limit, not your network's. The RB5009 with its faster quad-core ARM64 CPU handles container workloads much better. For production speed testing on fast networks, use dedicated hardware; for quick sanity checks and demos, the container works fine.

### Check Resource Usage

```
/system resource print
```

Compare free-memory to what you recorded earlier. Note the difference.

---

## Lab 5.3 — iperf3 Container

iperf3 provides detailed network performance testing. Unlike speedtest which runs in a browser, iperf3 requires a client application connecting to this server.

### Create Infrastructure

```
/interface/veth/add name=veth3 address=172.17.0.3/24 gateway=172.17.0.254 comment="iperf3"
/interface/bridge/port/add bridge=dockers interface=veth3 comment="iperf3"
/container/envs/add list=iperf3_envs key=TZ value="America/Denver"
/container/mounts/add list=iperf3_mount src=/usb1/iperf3 dst=/var/log comment="iperf3 logs"
```

### Deploy Container

```
/container/add remote-image=taoyou/iperf3-alpine interface=veth3 root-dir=/usb1/iperf3 mountlists=iperf3_mount envlists=iperf3_envs dns=172.17.0.254 start-on-boot=yes comment="iperf3"
```

> **Note:** We use `taoyou/iperf3-alpine` instead of `networkstatic/iperf3` because it provides ARM architecture support. The `networkstatic/iperf3` image is amd64-only.

### Start and Test

1. Start the container: select it and click **Start** under Actions, or via CLI:

   ```
   /container/start iperf3
   ```

2. From a device with iperf3 installed, test:

   ```
   iperf3 -c 172.17.0.3
   ```
> **Understanding your results:** Container-based iperf3 results reflect your router's CPU capacity, not your network speed. On an L009, expect ~500 Mbps with CPU usage around 85-90%. Every packet crosses from the physical interface through the router to the container bridge, and the container's network stack adds overhead. High retransmit counts (hundreds or more) are normal — that's the CPU falling behind momentarily and triggering TCP retransmissions. For higher throughput testing, consider the RB5009 or CCR2004, which have significantly more processing power for container workloads.

### Check Resource Usage

```
/system resource print
```

Note the cumulative memory impact of two containers.

---

## Lab 5.4 — nginx Content Server

nginx serves static content — HTML pages, PDFs, images, videos. Useful for:
- Landing pages for captive portals
- Documentation hosting
- File distribution at trade shows or events

### Known Issue: File Permissions

nginx runs as a non-root user by default and cannot read files on USB-mounted paths. This causes 403 Forbidden errors.

**Solution:** Create a custom nginx.conf that runs as root.

### Create Directory Structure

1. In WinBox, navigate to **Files** in the left menu.

2. Click on **usb1** to open it.

3. Click **New** in the upper left corner
    - Select **Directory**,
    - Name: `nginx-conf`
    - Directory: Click the drop-down, and then select **usb1**
    - Click **Select**
    - Click **Apply** & **OK**
    - Repeat for `nginx-content`

4. Alternately, you can create the nginx directories using CLI:

   ```
   /file/add name=usb1/nginx-conf type=directory
   /file/add name=usb1/nginx-content type=directory
   ```

### Create nginx.conf

5. On your computer, open a text editor (Notepad, VS Code, TextEdit, etc.).

6. Copy and paste the following configuration:

```nginx
user root;
worker_processes 1;
events { worker_connections 128; }
http {
    default_type application/octet-stream;
    types {
        text/html html htm;
        text/css css;
        application/javascript js;
        image/png png;
        image/jpeg jpg jpeg;
        image/svg+xml svg;
        application/pdf pdf;
        application/zip zip;
        video/mp4 mp4;
    }
    server {
        listen 80;
        location / {
            root /usr/share/nginx/html;
            index index.html;
            autoindex on;
        }
    }
}
```

> **Important:** Do not use macOS TextEdit for creating configuration or HTML files. TextEdit adds invisible formatting even in plain text mode. Use [BBEdit](https://www.barebones.com/products/bbedit/) (free mode), [VS Code](https://code.visualstudio.com/), or type `nano filename` in Terminal instead.

7. Save the file as `nginx.conf` (make sure your editor doesn't add `.txt` to the filename).

8. In WinBox Files, click **Upload** under Actions.

> ⚠️ **Read carefully.** If you get stuck here, re-read the steps above. The answer is in the instructions.

9. Select your `nginx.conf` file. It will upload to the root of the file system, not the folder you're viewing.

10. Drag the `nginx.conf` file from the root into the `usb1/nginx-conf/` folder.

> **Why no `include mime.types`?** MikroTik container mounts are directory-to-directory. When we mount our config directory over `/etc/nginx`, it replaces everything — including the default mime.types file. We define MIME types inline instead.

### Create Sample Content

11. On your computer, create a new file in your text editor.

12. Copy and paste the following HTML:

```html
<!DOCTYPE html>
<html>
<head>
    <title>MikroTik Content Server</title>
    <style>
        body { font-family: Arial, sans-serif; max-width: 800px; margin: 50px auto; padding: 20px; }
        h1 { color: #333; }
        .info { background: #f0f0f0; padding: 15px; border-radius: 5px; }
    </style>
</head>
<body>
    <h1>MikroTik Content Server</h1>
    <div class="info">
        <p>This page is served from an nginx container running on your MikroTik router.</p>
        <p>Add files to <code>usb1/nginx-content/</code> to serve them here.</p>
    </div>
</body>
</html>
```

13. Save the file as `index.html`.

14. In WinBox Files, click **Upload** and select your `index.html` file.

15. Drag the `index.html` file from the root into the `usb1/nginx-content/` folder.

### Create Infrastructure

16. Open a Terminal and run:

```
/interface/veth/add name=veth4 address=172.17.0.4/24 gateway=172.17.0.254 comment="nginx"
/interface/bridge/port/add bridge=dockers interface=veth4 comment="nginx"
```

### Create Mount Points

```
/container/mounts/add list=nginx_content src=/usb1/nginx-content dst=/usr/share/nginx/html comment="nginx content"
/container/mounts/add list=nginx_conf src=/usb1/nginx-conf dst=/etc/nginx comment="nginx config"
```

### Deploy Container

```
/container/add remote-image=library/nginx:latest interface=veth4 root-dir=/usb1/nginx mountlists=nginx_content,nginx_conf dns=172.17.0.254 logging=yes start-on-boot=yes comment="nginx content server"
```

### Start and Test

17. Navigate to **Containers** in the left menu.

18. Wait for the nginx container to finish downloading and extracting (watch the Log tab or check with `/container print`).

19. Select the nginx container and click **Start** under Actions, or via CLI:

    ```
    /container/start nginx
    ```

20. Verify it's running (flag shows **R**).

21. Open a browser and navigate to: **http://172.17.0.4**

22. You should see your sample page.

> ⚠️ **Read carefully.** If you get stuck here, re-read the steps above. The answer is in the instructions.

> **Troubleshooting:** If you get a 403 Forbidden error, verify the `nginx.conf` file is in `usb1/nginx-conf/` and contains `user root;` on the first line.

### Check Resource Usage

```
/system resource print
```

Three containers running — note the memory usage pattern.

> **Memory Reference:** On an L009 with 512 MiB RAM, running OpenSpeedTest, iperf3, and nginx together consumes approximately 5.5 MiB of container memory. Total system usage including RouterOS is approximately 157 MiB, leaving over 355 MiB free. Memory is not the limiting factor for containers on this device — CPU is.

---

## Lab 5.9 — Pi-hole DNS Ad Blocker (Optional)

*Prerequisites: Lab 5.1 (Container infrastructure), container network in LAN interface list*

Pi-hole is a network-wide DNS ad blocker with a web dashboard. This is an optional lab — RouterOS has a built-in AdList feature (Lab 28) that does the same job without a container. Pi-hole gives you a richer management interface and detailed query logging.

> **Note:** Pi-hole requires DNS access from the container network. Verify the container bridge is in the LAN interface list (Lab 5.1, step 3). If not:
> ```
> /interface/list/member/add list=LAN interface=dockers comment="Container network"
> ```

### Create Virtual Interface
```
/interface/veth/add name=veth-pihole address=172.17.0.5/24 gateway=172.17.0.254 comment="Pi-hole"
```
### Add to Container Bridge

```
/interface/bridge/port/add bridge=dockers interface=veth-pihole comment="Pi-hole"
```

### Deploy Container
```
/container/add remote-image=pihole/pihole:latest interface=veth-pihole root-dir=/usb1/pihole dns=172.17.0.254 start-on-boot=no comment="Pi-hole"
```

> **Note:** `start-on-boot=no` is intentional. Pi-hole uses significant resources during gravity updates. Only enable auto-start once you've verified it works.

### Start and Monitor

1. Wait for the image to download and extract. On USB 2.0 this may take 5-10 minutes.

2. Start the container:
```
/container/start [find comment="Pi-hole"]
```

3. Monitor the status:
```
/container/print
```

4. Pi-hole goes through three stages:
   - **S** (Stopped) — extracting or waiting to start
   - **C** (Starting with healthcheck) — gravity is downloading blocklists
   - **H** (Healthy) — ready to use

   The healthcheck stage can take several minutes as Pi-hole downloads and processes blocklists.

### Set Admin Password

5. Once the status shows **H** (Healthy), get a shell:
```
/container/shell [find comment="Pi-hole"]
```

6. Set the web interface password:
```
pihole setpassword
```

7. Enter and confirm your password, then type `exit` to leave the shell.

### Access the Dashboard

8. Open a browser and navigate to `http://172.17.0.5/admin`

9. Log in with the password you just set.

10. The dashboard shows:
    - **Total Queries** — DNS queries processed
    - **Queries Blocked** — ads and trackers caught
    - **Domains on Lists** — should show ~78,000+ from the default blocklist

### Using Pi-hole as Your DNS Server

To route DNS queries through Pi-hole for ad blocking, point your DHCP server's DNS setting at the Pi-hole container IP:
```
/ip/dhcp-server/network/set [find comment="VLAN 20"] dns-server=172.17.0.5
```

> **Caution:** If the Pi-hole container stops, DNS resolution stops for any network pointing at it. Keep the router's DNS available as a fallback, or only point test VLANs at Pi-hole until you're confident in the setup.

### Cleanup (if removing)

If you want to remove Pi-hole:

```
/container/stop [find comment="Pi-hole"]
/container/remove [find comment="Pi-hole"]
/interface/bridge/port/remove [find interface=veth-pihole]
/interface/veth/remove veth-pihole
```

---

## Container Compatibility Notes

### Containers Tested and Working (RouterOS 7.24.4, L009)

| Container | Image | Status |
|-----------|-------|--------|
| OpenSpeedTest | openspeedtest/latest | ✅ Working (Lab 5.2) |
| iperf3 | taoyou/iperf3-alpine | ✅ Working (Lab 5.3) |
| nginx | library/nginx:latest | ✅ Working (Lab 5.4) |
| Pi-hole | pihole/pihole:latest | ✅ Working (Lab 5.9) |

### Containers That Don't Work

**freeRADIUS** — The official `freeradius/freeradius-server` image failed on ARM 32-bit devices (L009) with an architecture mismatch. It may work on ARM 64-bit devices (RB5009, CCR2004) but has not been tested. For RADIUS authentication on MikroTik, use the built-in User Manager feature instead (covered in Lab 12).

**hEX S refresh (2025)** — Despite being an ARM 32-bit device with container support enabled, the hEX S refresh (EN7562CT CPU) fails to run most container images — including ARM32-native builds like `arm32v7/nginx:alpine`. Containers download and extract successfully but crash immediately with "Illegal instruction" (signal 4). The CPU does not support all ARMv7 instructions that standard container binaries expect. If you need containers, use the L009 or RB5009 instead.

### Alternatives

- For DNS-based ad blocking without containers, RouterOS 7.15+ includes a built-in **AdList** feature. It works on all MikroTik hardware including older MIPSBE devices. See Lab 28 for configuration details.
- For RADIUS, use RouterOS User Manager (covered in Lab 12)

## Container Troubleshooting

### Container Status Flags

When viewing containers with `/container print`, the flag column shows container state. These same flags appear in the first column of the Container list in WinBox.

| Flag | Meaning |
|------|---------|
| R | Running |
| S | Stopped |
| F | Download/Extract Failed |
| E | Extracting |
| N | Starting |
| C | Starting with Healthcheck |
| U | Unhealthy |

> **Referencing containers:** Throughout this guide, we use the `[find comment="..."]` syntax to target containers — for example, `/container/stop [find comment="Pi-hole"]`. This works reliably because the comment is something you set explicitly when creating the container. Index numbers (`/container/stop 0`) work for quick one-offs but change when containers are added or removed. Container names are derived from the image and can be unpredictable — OpenSpeedTest shows up as `latest`, not `openspeedtest`.

### Container Won't Start After Reboot

Containers with `start-on-boot=yes` may fail if the USB drive isn't mounted yet when RouterOS tries to start them.

**Solution:** Verify USB is mounted, then manually start:

```
/disk print
/container/start [find comment="nginx content server"]
```

### 403 Forbidden on nginx

The nginx worker process can't read files on USB-mounted paths.

**Solution:** Use the custom nginx.conf with `user root;` as shown in Lab 5.4.

### Container Status Shows "error"

Check the container log:

```
/log print where topics~"container"
```

Enable container logging if not already enabled:

```
/container/set [find comment="OpenSpeedTest"] logging=yes
```

### "Illegal instruction" or "Signal 4" Error

This indicates a CPU architecture mismatch. The container image was built for a different architecture than your router supports.

**Solution:** This typically affects the hEX S refresh (EN7562CT CPU), which does not support all ARMv7 instructions that standard container binaries expect. There is no known workaround. Use an L009, RB5009, or hAP ax2 instead for container workloads.

---

Once you make it to this point, you can advance to Lab 06.
