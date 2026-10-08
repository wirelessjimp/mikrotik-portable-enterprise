# Lab 12 — Traffic Analysis

*Prerequisites: Lab 1 (your L009), Lab 7 (the enterprise Wi-Fi and your `user2` login), Lab 9 if you want to watch EAP-TLS. Wireshark is installed on your laptop.*

**Why:** When something doesn't work, you need to see what's on the wire. Torch shows who is talking right now. The packet sniffer captures the packets themselves, so Wireshark can decode them. You use both on traffic you built in earlier labs: your own pings first, then the RADIUS conversation between your mAP and your L009.

> ### ⚠️ STOP AND READ
> A capture holds whatever crossed the wire, and that can include passwords. Delete your capture files from the router and from your laptop when you finish (Lab 18.2).

### 12.1 Torch: see your own traffic

1. **L009 window:** click **Tools**, then **Torch**.
2. Set **Interface** to `bridge`. Your laptop's backdoor cable is in this bridge. Leave the other fields empty. Click **Start**.
3. In a terminal on your laptop, run `ping 192.168.88.1`. Rows appear in Torch for **ICMP**, with your laptop's address and `192.168.88.1`. Write down your laptop's address.

   > **Why:** Torch counts traffic as it passes an interface. It is the quickest way to answer "is anything reaching this interface?"

4. Press **Ctrl+C** in the laptop terminal to stop the ping, then click **Stop** in Torch.

### 12.2 Torch: watch the RADIUS requests

5. In Torch, set **Interface** to `wg-server`, **Protocol** to `udp`, and **Port** to `1812`. Click **Start**.

   > **Why:** Your mAP sends its RADIUS requests through the WireGuard tunnel from Lab 6, so you watch the tunnel interface, not a VLAN.

6. On your phone, join `Student31-EAP` as in Lab 7 step 5 and sign in as `user2`. Use your own label.
7. A row appears with **Src. Address** `10.255.255.2`, **Dst. Address** `10.255.255.1`, and port `1812`. These are your mAP and your L009's tunnel addresses.
8. Click **Stop**.

   > **Note:** If you see the requests but the login fails, the problem is the RADIUS settings, not the network path. If you see nothing, the requests aren't reaching your L009.

### 12.3 Capture to a file and open it in Wireshark

9. **L009 window:** click **Tools**, then **Packet Sniffer**. On the **General** tab, click the **+** next to **File Name** and type `Student31-ping.pcapng`. Set **File Limit** to `1000`.
10. Click the **Filter** tab. Click the **+** next to **Interfaces** and select `bridge`. Click **Apply**.
11. Click **Start**. Run `ping 192.168.88.1` on your laptop for a few seconds, then press **Ctrl+C**. Click **Stop**.
12. Click **Files**, select `Student31-ping.pcapng`, and click **Download...** under **Actions**.
13. Open the file in Wireshark. Type `icmp` in the display filter bar and press **Enter**. The echo requests and replies from your ping are listed.

### 12.4 Stream the RADIUS conversation into Wireshark

14. In the packet sniffer, click the **Filter** tab. Remove `bridge` and add `wg-server`. Click the **Streaming** tab, check **Streaming Enabled**, and set **Server** to your laptop's address from step 3. Leave **Port** at `37008`. Click **Apply**. Don't click **Start** yet.
15. In Wireshark, on the main screen, scroll down to **UDP Listener remote capture** and click the gear next to it. Set **Listen port** to `37008` and **Payload type** to `tzsp`. Click **Start**.

   > **Note:** The payload type is case-sensitive. `TZSP` in capital letters makes Wireshark decode the stream wrongly.

16. In WinBox, click **Start** on the packet sniffer.
17. On your phone, join `Student31-EAP` again and sign in as `user2`.
18. In Wireshark, type `radius` in the display filter bar and press **Enter**. Find an `Access-Request`, then the `Access-Challenge` packets that follow it, then an `Access-Accept`. Click the `Access-Request` and open its attributes. You see the **User-Name** your phone sent.
19. Click **Stop** in WinBox and in Wireshark. Remove `Student31-ping.pcapng` from **Files**.

   > **Why:** You are watching both sides of a RADIUS login, the way the AP and the server saw it. This is the first place to look when an enterprise login fails.
