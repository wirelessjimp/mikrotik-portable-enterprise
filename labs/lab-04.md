# Lab 4 — Containers

*Prerequisites: Lab 1 (WAN connected), Lab 2 (including the device-mode check). The container package, device mode, and USB drive are set up by the instructor before class.*

**Why:** Containers let your router run small programs of its own. You'll build three: a speed test server, an iperf3 server, and a web server. The first you build by filling in a form. The other two you build by pasting one command each. *(First draft of this paragraph.)*

### 4.1 OpenSpeedTest (in the form)

1. Click **Container** in the left menu, then **New**.

   > ### ⚠️ STOP AND READ
   > Some fields show only a **+** button. Click the **+** first, and a box appears. Type the container image in **Remote Image**, not in **Name**.

2. On the **General** tab, set these fields in this order:
   - **Comment:** `OpenSpeedTest`
   - **Name:** `SpeedTest`
   - **Remote Image:** click the **+**, then type `openspeedtest/latest`
   - **Check Certificate:** uncheck it
   - **Root Dir:** `usb1/speedtest`
   - **Interface:** click the **+** and choose `veth-speedtest` from the list
   - **DNS:** click the **+** and type `172.17.0.254`
   - **Mountlists:** click the **+** and choose `speedtest_mount` from the list
   - **Envlists:** click the **+** and choose `speedtest_envs` from the list
   - **Start On Boot:** check it
3. Click **Apply**.

   > ### ⚠️ STOP AND READ
   > Uncheck **Check Certificate** before you click **Apply**. Once you click **Apply**, the router starts downloading and the checkbox disappears from the window. If you leave it checked, the download stops with `check registry failed: SSL: ssl: crl not found for: "CN=*.docker.com"`.

4. Watch the red bar at the top of the dialog. It counts through the image's layers, for example `downloading layer 15/29`. The red doesn't mean an error.
5. Wait until the badge at the bottom changes to **STOPPED**. The container is downloaded and ready, and it isn't running yet.

   > **Note:** The download took about 90 seconds on a fast home connection. In class it may take longer, because everyone is pulling at once. Don't click **Apply** again while it's working.

6. Click **Start** in the **Actions** list on the right of the dialog. The badge at the bottom changes to **RUNNING**, and the flag in the container list changes to **R**.

   > **Note:** The first column of the container list shows a flag for the container's state. **E** is downloading and extracting, **S** is stopped, and **R** is running. If you see **F**, the download failed. The full list is at the end of this lab.

7. Open a browser and go to `http://172.17.0.2:3000`. The OpenSpeedTest page loads.

   > **Note:** If the page doesn't load, check that the container's flag in the list is **R** and that the Ethernet cable is in **ether7**.

8. Open the speed test in its own browser window, not just a tab. Make the window smaller and drag it over WinBox, so the CPU reading in the status bar at the bottom of WinBox stays visible.
9. Click **Start**. Watch the **CPU** percentage in the WinBox status bar while the test runs.
10. When the test says **All done**, write down your download and upload numbers. The CPU drops back to near 0%.

    > **Why:** The speed test runs inside your router, so the router's own processor is doing the work and the limit you see is the L009's, not your network's. Don't expect the numbers to match anyone else's.

### 4.2 iperf3 (one command)

11. Click **New Terminal** in the left menu. Copy this command, paste it into the Terminal, and press **Enter**:

```
/container/add name=iperf3 comment=iperf3 remote-image=taoyou/iperf3-alpine check-certificate=no root-dir=usb1/iperf3 interface=veth-iperf3 dns=172.17.0.254 mountlists=iperf3_mount envlists=iperf3_envs start-on-boot=yes
```

12. Type `/container/print` and press **Enter**. Repeat until the new `iperf3` entry shows an **S** flag. It's downloaded and ready.
13. Start it:

```
/container/start [find comment="iperf3"]
```

14. Run `/container/print` again. The `iperf3` entry now shows an **R** flag.

> **Why:** You just built a container by filling in a form. This is the same kind of container as one line of text. Each `name=value` pair in the command matches a field in the form.

> **Note:** The image is `taoyou/iperf3-alpine` because `networkstatic/iperf3` is amd64-only and the L009 is `arm`.

### 4.3 nginx web server

15. In **New Terminal**, run these two commands:

```
/file/add name=usb1/nginx-conf type=directory
/file/add name=usb1/nginx-content type=directory
```

> **Why:** The container reads its settings from `nginx-conf` and its web files from `nginx-content`. The router creates the log folders itself when a container starts, but these two have to exist before you put files in them.

16. Open a browser and type `http://172.18.0.2`. Don't use `portal.lab`. In the **Downloads** card, click **nginx.conf**.

    > ### ⚠️ STOP AND READ
    > Your browser may block this download as insecure, the same as with Lab Notes. Look near the address bar and click **Keep** before the pop-up disappears.

17. In WinBox, click **Files**, then **Upload...** in the **Actions** list, and choose the file from your **Downloads** folder. The file appears at the top level, outside `usb1`. Drag it onto `usb1/nginx-conf` and drop it there.

    > ### ⚠️ STOP AND READ
    > An upload always lands at the top of the Files list, even when a folder is highlighted. Drag it in yourself.

18. Expand `usb1/nginx-conf`. The file must be named exactly `nginx.conf`. If it says something else, such as `nginx (1).conf`, double-click the file's name. A **File** window opens, and its **Name** field shows the whole path, for example `usb1/nginx-conf/nginx (1).conf`. Change only the last part so the path reads `usb1/nginx-conf/nginx.conf`, then click **Apply** and then **OK**.

    > ### ⚠️ STOP AND READ
    > Don't replace the whole path with just `nginx.conf`. The path is the folder, so that would move the file out of `usb1/nginx-conf`.

19. In **New Terminal**, paste this command and press **Enter**:

```
/container/add name=nginx comment=nginx remote-image=library/nginx:latest check-certificate=no root-dir=usb1/nginx interface=veth-nginx dns=172.17.0.254 mountlists=nginx_content,nginx_conf logging=yes start-on-boot=yes
```

20. Run `/container/print` until the `nginx` entry shows an **S** flag.
21. Start it:

```
/container/start [find comment="nginx"]
```

22. Run `/container/print` again. The `nginx` entry now shows an **R** flag.
23. Open a browser and go to `http://172.17.0.4`. The page says **403 Forbidden**, with `nginx` under it.

    > **Note:** This is expected. nginx is running, and it has no web page to show yet, because `usb1/nginx-content` is empty.

24. Open `http://172.18.0.2` again. In the **Downloads** card, click **Sample web page**. Click **Keep** if your browser blocks it.
25. In WinBox, click **Files**, then **Upload...**, and choose the downloaded `index.html`. When it appears at the top level, drag it onto `usb1/nginx-content`.
26. Reload `http://172.17.0.4`. The page says **It works.**, one line at a time.

> **Make it yours:** The sample page has a `NAME` line near the bottom of its script. Open the file in a plain text editor (not TextEdit in rich-text mode) and change it. The upload-again steps are not written yet. See the open items.

### Container status flags

| Flag | Meaning |
|------|---------|
| R | Running |
| S | Stopped |
| F | Download/Extract Failed |
| E | Extracting |
| N | Starting |
| C | Starting with Healthcheck |
| U | Unhealthy |

> **Referencing containers:** Container names come from the image and can be unpredictable, so the CLI examples target a container by its comment, for example `/container/start [find comment="iperf3"]`. Index numbers such as `/container/start 2` also work, but they change when containers are added or removed.
