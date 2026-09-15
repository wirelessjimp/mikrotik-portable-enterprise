# Lab 20 — Traffic Analysis & Troubleshooting

*Prerequisites: Lab 1 (Initial Configuration)*

When troubleshooting network issues, you need to see what's actually happening on the wire. MikroTik provides two tools for this: Packet Sniffer for full packet captures, and Torch for real-time traffic monitoring.

---

## Lab 20.1 — Packet Sniffer (Wireshark Capture)

The MikroTik Packet Sniffer captures traffic on any interface and saves it in PCAP format for analysis in Wireshark.

### Basic Capture to File

1. In WinBox, navigate to **Tools** → **Packet Sniffer**

2. Configure the capture:
   - **Interface:** Select an interface to capture (e.g., vlan255bridge)
   - **File Name:** Click the arrow, enter a name (e.g., `capture1`)
   - **File Limit:** Set a size limit if desired (e.g., 1000 KiB)

3. Optional: Set pre-capture filters to limit what's captured:
   - **Filter Stream:** Check to enable filtering
   - **Filter Protocol:** IP, IPv6, etc.
   - **Filter Port:** Specific port number
   - **Filter IP Address:** Source or destination IP

4. Click **Apply**

5. Click **Start** to begin capturing

6. Generate some traffic (ping, browse, etc.)

7. Click **Stop** to end the capture

### View Captured Packets

8. Click the **Packets** tab to view captured packets in the MikroTik interface

9. This view is limited — for full analysis, download the file

### Download and Analyze in Wireshark

10. Navigate to **Files** in the left menu

11. Find your capture file (e.g., `capture1.pcap`)

12. Right-click and select **Download** (or drag to your desktop)

13. Open the file in Wireshark for full analysis

---

## Lab 20.2 — Live Streaming to Wireshark

Instead of capturing to a file, you can stream packets directly to Wireshark in real-time. This is useful for live troubleshooting.

### Configure MikroTik for Streaming

1. Navigate to **Tools** → **Packet Sniffer**

2. Configure:
   - **Interface:** Select the interface to capture
   - **Streaming Enabled:** ✓ Checked
   - **Server:** Enter your computer's IP address
   - **Port:** 37008 (default)

3. Click **Apply** (don't click Start yet)

### Configure Wireshark to Receive

4. Open Wireshark on your computer

5. In the main interface list, scroll down to find **UDP Listener remote capture**

6. Click the gear icon next to it to configure:
   - **Listen port:** 37008
   - **Payload type:** TZSP

7. Click **Start** to begin listening

### Start the Stream

8. Return to WinBox and click **Start** on the Packet Sniffer

9. Wireshark will display packets in real-time as they cross the MikroTik interface

10. When finished, click **Stop** in WinBox, then stop the capture in Wireshark

> **Use case:** Live streaming is perfect for troubleshooting intermittent issues — you see packets as they happen without filling up storage on the router.

---

## Lab 20.3 — Torch (Real-Time Traffic Monitor)

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

## Lab 20 Summary

| Tool | Use Case | Output |
|------|----------|--------|
| Packet Sniffer (file) | Detailed analysis, save for later | PCAP file for Wireshark |
| Packet Sniffer (stream) | Live troubleshooting | Real-time Wireshark display |
| Torch | Quick "is traffic flowing?" check | Real-time rates and addresses |

---
