# Lab 21 — Traffic Analysis & Troubleshooting

*Prerequisites: Lab 1 (Initial Configuration)*

When troubleshooting network issues, you need to see what's actually happening on the wire. MikroTik provides two tools for this: Packet Sniffer for full packet captures, and Torch for real-time traffic monitoring.

---

## Lab 21.1 — Packet Sniffer (Wireshark Capture)

The MikroTik Packet Sniffer captures traffic on any interface and saves it in PCAP format for analysis in Wireshark.

### Basic Capture to File

1. In WinBox, navigate to **Tools** → **Packet Sniffer**

2. On the **General** tab:
   - **File Name:** Click the **+** and enter a name with the `.pcapng` extension (e.g., `capture1.pcapng`)
   - **File Limit:** Set a size limit (e.g., `1000` kb)
   - Leave Memory Limit and other settings at defaults

3. Click the **Filter** tab to narrow what's captured:
   - **Interfaces:** Click **+** and select the interface to capture on (e.g., `ether1`, `vlan20`, `bridge`)
   - **IP Address:** Click **+** to filter by a specific IP
   - **Port:** Click **+** to filter by port number
   - **Direction:** Select `rx`, `tx`, or leave as `any`

   > **Tip:** Leave all filters empty to capture everything. Add filters when you know what you're looking for and want to reduce noise.

4. Click **Apply**

5. Click **Start** under Actions on the right

6. Generate some traffic (ping, browse, etc.)

7. Click **Stop** to end the capture

### View Captured Packets

8. Click the **Packets** button on the right-hand side to view captured packets in the MikroTik interface

9. This view is limited — for full analysis, download the file

### Download and Analyze in Wireshark

10. Navigate to **Files** in the left menu

11. Find your capture file (e.g., `capture1.pcapng`)

12. Click and select **Download** 

13. Open the file in Wireshark for full analysis

---

## Lab 21.2 — Live Streaming to Wireshark

Instead of capturing to a file, you can stream packets directly to Wireshark in real-time. This is useful for live troubleshooting.

### Configure MikroTik for Streaming

1. Navigate to **Tools** → **Packet Sniffer**

2. Click the **Filter** tab:
   - **Interfaces:** Click **+** and select the interface to capture

3. Click the **Streaming** tab:
   - **Streaming Enabled:** ✓ Checked
   - **Server:** Enter your computer's IP address
   - **Port:** 37008 (default)

4. Click **Apply** (don't click Start yet)
   
### Configure Wireshark to Receive

4. Open Wireshark on your computer

5. In the main interface list, scroll down to find **UDP Listener remote capture**

6. Click the gear icon next to it to configure:
   - **Listen port:** 37008
   - **Payload type:** tzsp
   >**NOTE:** Payload type is case-sensitive. If you type `TZSP` instead of `tzsp` Wireshark won't decode the dump correctly.

7. Click **Start** to begin listening

### Start the Stream

8. Return to WinBox and click **Start** on the Packet Sniffer

9. Wireshark will display packets in real-time as they cross the MikroTik interface

10. When finished, click **Stop** in WinBox, then stop the capture in Wireshark

> **Use case:** Live streaming is perfect for troubleshooting intermittent issues — you see packets as they happen without filling up storage on the router.

---

## Lab 21.3 — Torch (Real-Time Traffic Monitor)

Torch shows traffic flowing through an interface in real-time. Think of it as a quick "is traffic flowing?" check without capturing full packets.

### What Torch Shows

Torch displays:
- Source and destination addresses
- Protocol
- Port numbers
- Transmit and receive rates
- VLAN IDs

### When to Use Torch

- **Quick check:** Is traffic reaching this interface?
- **Pre-firewall view:** Torch sees traffic before firewall rules apply
- **Bandwidth monitoring:** Who's using bandwidth right now?
- **RADIUS troubleshooting:** Is the AP actually sending RADIUS requests?

### Basic Torch Usage

1. Navigate to **Tools** → **Torch**

2. Configure:
   - **Interface:** Select the interface to monitor
   - **Src. Address:** Filter by source IP (optional)
   - **Dst. Address:** Filter by destination IP (optional)
   - **Port:** Filter by port number (optional)
   - **Protocol:** Filter by protocol (optional)
   - **Entry Timeout:** 00:00:10 (10 seconds to allow for easier reading)

3. Click **Start**

4. Traffic appears in real-time, showing:
   - **Src. Address** and **Dst. Address**
   - **Protocol** (TCP, UDP, ICMP, etc.)
   - **Src. Port** and **Dst. Port**
   - **Tx Rate** and **Rx Rate**

5. Click **Stop** when finished

### Example: Verify RADIUS Traffic

To verify an AP is sending RADIUS requests:

1. Open Torch on **vlan255bridge** (or your management VLAN)

2. Set **Port:** 1812

3. Set **Protocol:** UDP

4. Click **Start**

5. Attempt a wireless authentication

6. You should see UDP traffic from the AP's IP to 10.10.255.1 on port 1812

If you see the requests but authentication fails, the issue is RADIUS configuration, not connectivity.

### Example: Check DHCP Traffic

To verify DHCP requests are reaching the router:

1. Open Torch on the appropriate bridge interface

2. Set **Port:** 67,68

3. Set **Protocol:** UDP

4. Click **Start**

5. Connect a client or release/renew DHCP

6. You should see DHCP discover/request packets

---

## Lab 21 Summary

| Tool | Use Case | Output |
|------|----------|--------|
| Packet Sniffer (file) | Detailed analysis, save for later | PCAP file for Wireshark |
| Packet Sniffer (stream) | Live troubleshooting | Real-time Wireshark display |
| Torch | Quick "is traffic flowing?" check | Real-time rates and addresses |

---
