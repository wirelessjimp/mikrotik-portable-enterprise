# Lab 20 — Graphing and Bandwidth Test

*Prerequisites: Lab 1 (your L009). The test between your two devices needs the tunnel from Lab 6.*

Two ways to measure what your router is doing: graphs of traffic over time, and a throughput test between your own devices.

---

## Lab 20.1 — Interface Graphing

Your L009 already graphs three interfaces, `ether1`, `ether8`, and `vlan255`. The instructor's script turned them on.

1. Click **Tools**, then **Graphing**.
2. Click the **Interface Graphs** tab. The three interfaces are listed.
3. Double-click `ether1`. The graph shows the traffic on your WAN port. Your router has only been running for a short while, so the lines are short.
4. The window offers four views:
   - **Daily:** the last 24 hours
   - **Weekly:** the last 7 days
   - **Monthly:** the last 30 days
   - **Yearly:** the last 365 days
5. Close the graph.

### Graph another interface

6. In the **Interface Graphs** tab, click **New**. Set **Interface** to `bridge`, which holds your backdoor cable. Click **Apply**, then **OK**.
7. Generate some traffic from your laptop, such as a large download, and open the new graph.

### Real-time numbers

For rates right now, not history:

8. Click **Interfaces**, double-click an interface, and open the **Traffic** section at the bottom. The transmit and receive rates update live.

---

## Lab 20.2 — Bandwidth Test

Test throughput to a public server, then between your own devices.

### Test to a public server

1. Click **Tools**, then **Bandwidth Test**.
2. Set:
   - **Test To:** `mikrotik.speedtest.alagas.net`
   - **Protocol:** TCP
   - **Direction:** both
   - **Username:** `speedtest`
   - **Password:** `MikroTikSG`
3. Click **Start**. The results show throughput in both directions.

> **Note:** This is a community-run server, so it may not answer. The original MikroTik public server was shut down in 2025. Your whole class shares one internet connection, so expect numbers lower than the connection can do.

### Test between your two devices

This test measures the link and the devices, without the internet in the way.

4. **L009 window:** click **Tools**, then **BTest Server**. Check **Enabled**, and uncheck **Authenticate**. Click **Apply**, then **OK**.
5. **mAP window:** click **Tools**, then **Bandwidth Test**. Set:
   - **Test To:** `10.255.255.1`, your L009's end of the tunnel
   - **Protocol:** TCP
   - **Direction:** both
6. Click **Start**. Write down the result.

   > **Why:** The mAP reaches the L009 through the WireGuard tunnel, so this measures the tunnel. The mAP's Ethernet ports run at 100 Mbit/s, so the result can't pass that.

7. If your mAP is on the 15 cm jumper, run it again with **Test To** set to `10.10.255.1`, your L009's address on the jumper network, and compare the two results.
8. When you finish, turn the server off again: **L009 window:** click **Tools**, then **BTest Server**, uncheck **Enabled**, and click **Apply**.

---

## Lab 20 Summary

| Tool | Purpose |
|------|---------|
| Interface Graphing | Traffic over time, per interface |
| Bandwidth Test | Throughput to a server, or between your own devices |
