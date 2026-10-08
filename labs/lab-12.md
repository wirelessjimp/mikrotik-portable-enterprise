# Lab 12 — Traffic Analysis

*Prerequisites: Lab 1 (your L009), Lab 7 (the enterprise Wi-Fi and your `user2` login), Lab 9 if you want to watch EAP-TLS. Wireshark is installed on your laptop.*

**Why:** When something doesn't work, you need to see what's on the wire. Torch shows who is talking right now. The packet sniffer captures the packets themselves, so Wireshark can decode them. You use both on traffic you built in earlier labs: your own pings first, then the RADIUS conversation between your mAP and your L009.

> ### ⚠️ STOP AND READ
> A capture holds whatever crossed the wire, and that can include passwords. Delete your capture files from the router and from your laptop when you finish (Lab 18.2).

### 12.1 Torch: see your own traffic

1. **L009 window:** click **Tools**, then **Torch**.
2. Set **Interface** to `bridge`. Your laptop's backdoor cable is in this bridge. Leave the other fields as they are. Click **Start**.
3. In a terminal on your laptop, run `ping 192.168.88.1`. Rows appear in Torch for **ICMP**, with your laptop's address and `192.168.88.1`. Write down your laptop's address. Your open WinBox sessions show up too, on port `8291`, so look for the row whose **Protocol** reads `1 (icmp)`.

   > **Why:** Torch counts traffic as it passes an interface. It is the quickest way to answer "is anything reaching this interface?"

4. Press **Ctrl+C** in the laptop terminal to stop the ping, then click **Stop** in Torch.

### 12.2 Torch: watch the RADIUS requests

5. In Torch, set **Interface** to `wg-server`. Set **Protocol** to `udp` from the list, and type `1812` in **Port**. Set **Entry Timeout** to `00:00:15`, so rows stay on screen long enough to read. Click **Start**.

   > **Why:** Your mAP sends its RADIUS requests through the WireGuard tunnel from Lab 6, so you watch the tunnel interface, not a VLAN.

6. On your phone, join `Student31-EAP` as in Lab 7 step 5 and sign in as `user2`. Use your own label.
7. Rows appear with **Src.** `10.255.255.2` and **Dst.** `10.255.255.1:1812 (radius)`. You may see several, one for each request, each from a different source port. These are your mAP's and your L009's tunnel addresses.
8. Click **Stop**.

   > **Note:** If you see the requests but the login fails, the problem is the RADIUS settings, not the network path. If you see nothing, the requests aren't reaching your L009.

### 12.3 Capture to a file and open it in Wireshark

9. **L009 window:** click **Tools**, then **Packet Sniffer**. On the **General** tab, click the **+** next to **File Name** and type `Student31-ping.pcapng`. Leave **File Limit** at `1000`.
10. Click the **Filter** tab. Click the **+** next to **Interfaces** and select `bridge`. Click the **+** next to **IP Protocol** and select `icmp` from the list. Click **Apply**.

   > **Why:** Your laptop sends and receives other traffic all the time. Without the `icmp` filter, the capture fills its `1000` kb file in a few seconds, before your ping, and the file holds your web traffic too.
11. Under **Actions** on the right, click **Start**. Run `ping 192.168.88.1` on your laptop for a few seconds, then press **Ctrl+C**. Under **Actions**, click **Stop**. In **Files**, `Student31-ping.pcapng` is only a few KiB. In the sniffer, **Packets** under **Configuration** lists your pings in pairs: **rx** from your laptop to `192.168.88.1`, then **tx** back, `98` bytes each.
12. Click **Files**, select `Student31-ping.pcapng`, and click **Download...** under **Actions**.
13. Open the file in Wireshark. The packets are your pings, as pairs of `Echo (ping) request` and `Echo (ping) reply`. The **Info** column says which reply answers which request, for example `(reply in 2)`. The capture filter already kept everything else out, so you don't need a display filter here.

### 12.4 Stream the RADIUS conversation into Wireshark

14. In the packet sniffer, click the **Filter** tab. Remove `bridge` and `icmp`. Under **Interfaces**, add `wg-server`. Under **IP Protocol**, pick `udp`, and under **Port**, type `1812`. Click the **Streaming** tab, check **Streaming Enabled**, and set **Server** to your laptop's address from step 3. Leave **Port** at `37008`. Click **Apply**. Don't click **Start** yet.
15. In Wireshark, on the main screen, scroll down to **UDP Listener remote capture** and click the gear next to it. Set **Listen port** to `37008` and **Payload type** to `tzsp`. Click **Start**.

   > **Note:** The payload type is case-sensitive. `TZSP` in capital letters makes Wireshark decode the stream wrongly.

16. In WinBox, under **Actions** in the packet sniffer, click **Start**.
17. On your phone, join `Student31-EAP` again. Sign in as `user2`, or with your certificate from Lab 9.
18. Packets appear in Wireshark as your phone signs in. The **Protocol** column reads `RADIUS`, and the **Info** column shows the login itself: `Response, Identity`, then a run of TLS messages such as `Client Hello`, `Server Hello, Certificate`, and `Change Cipher Spec`, then `Success`. The capture filter already kept everything else out, so you don't need a display filter. Click a packet and open **RADIUS** in the details pane. The **Code** reads `Access-Request` for packets from your mAP, `Access-Challenge` for your L009's replies, and `Access-Accept` for the final `Success`.
19. Click **Stop** in WinBox and in Wireshark. Remove `Student31-ping.pcapng` from **Files**.

   > **Why:** You are watching both sides of a RADIUS login, the way the AP and the server saw it. This is the first place to look when an enterprise login fails.
