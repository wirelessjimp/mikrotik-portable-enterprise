# Lab 20 — Graphing and Bandwidth Test

*Prerequisites: Lab 1 (your L009).*

Two ways to measure what your router is doing: graphs of traffic over time, and a throughput test to a remote server.

---

## Lab 20.1 — Interface Graphing

Your L009 already graphs three interfaces, `ether1`, `ether8`, and `vlan255`. The instructor's script turned them on.

1. Click **Tools**, then **Graphing**. The window has tabs for **Interface Rules**, **Queue Rules**, and **Resource Rules**, and a matching graphs tab for each.
2. On the **Interface Rules** tab, three rows are listed: `ether1`, `ether8`, and `vlan255`. Each shows **Allow Address** `0.0.0.0/0` and **Store on Disk** `yes`. Each row is a rule that tells the router to keep a graph for that interface.
3. Click the **Interface Graphs** tab. The same three interfaces are listed.
4. Double-click `ether1`. The **Interface Graph** window opens with four views:
   - **Daily:** the last 24 hours
   - **Weekly:** the last 7 days
   - **Monthly:** the last 30 days
   - **Yearly:** the last 365 days

   The **Daily** graph shows **Rx** and **Tx** for your WAN port, with the current rates under it.
5. Close the graph with its **X**.

### See the same graphs without logging in

6. In your laptop's browser, open `https://192.168.88.1/graphs/iface/ether1/`. Your browser may show a certificate warning. The page shows all four graphs, with the maximum, average, and current rates under each. You didn't log in.

   > **Note:** Anyone who can reach your router's web page can see these graphs, because **Allow Address** is `0.0.0.0/0`. To limit that, double-click a rule on the **Interface Rules** tab and change **Allow Address** to your own network.

### Graph another interface

7. On the **Interface Rules** tab, click **New**. Set **Interface** to `bridge`, which holds your backdoor cable. Click **Apply**, then **OK**.
8. Click the **Interface Graphs** tab. `bridge` is listed at once, but its graph is empty. Data takes five to ten minutes to appear. Do the next section while you wait.

### Real-time numbers

For rates right now, not history:

9. Click **Interfaces**, then double-click `bridge`. WinBox shows each interface's comment as a heading above it, here **Backdoor bridge**. Click the **Traffic** tab. **Tx/Rx Rate** and the **Byte Graph** below it update live.
10. Generate some traffic from your laptop, such as a large download or a continuous ping, and watch **Tx/Rx Rate** move.
11. After five to ten minutes, open the `bridge` graph again. It now has data.

---

## Lab 20.2 — Bandwidth Test

Test the throughput from your L009 to a remote server.

1. Click **Tools**, then **Bandwidth Test**.
2. Set:
   - **Address:** `mikrotik.speedtest.alagas.net`
   - **Protocol:** `tcp`
   - **Direction:** `both`
   - **User:** `speedtest`
   - **Password:** `MikroTikSG`
3. Click **Start**. The status line reads `running...`.
4. Scroll down to the results. **Tx/Rx Current**, **Tx/Rx 10s Average**, and **Tx/Rx Total Average** show the speed in each direction, with a graph below them. **Tx** is what your L009 sends, and **Rx** is what it receives. On the instructor's L009, the total average was about `38 Mbps` up and `140 Mbps` down.
5. Look at **Local CPU Load** and **Remote CPU Load**. On the instructor's L009 they read `77 %` and `12 %`.
6. Click **Stop** to end the test.

   > **Why:** The test runs from your L009 out of its WAN port, so it measures your kit's connection to that server. Wi-Fi isn't in the path. If **Local CPU Load** gets close to `100 %`, your router is the limit, not the connection.

> **Note:** This is a community-run server, so it may not answer. The original MikroTik public server was shut down in 2025. Your whole class shares one internet connection, so expect numbers lower than the connection can do.

---

## Lab 20 Summary

| Tool | Purpose |
|------|---------|
| Interface Graphing | Traffic over time, per interface |
| Bandwidth Test | Throughput from your L009 to a remote server |
