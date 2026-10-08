# Lab 21 — Time and NTP

*Prerequisites: Lab 1 (your L009) and Lab 2, where steps 11 to 15 check your router's clock, its time servers, and the **Peers** list.*

Accurate time matters for logs, certificates, and scheduled tasks. Lab 2 showed where your router gets its time. Here you look at how the script set that up, and make your L009 hand out time to its own network.

---

## Lab 21.1 — Your NTP client

Your L009 is already an NTP client. The instructor's script turned it on.

1. Click **System**, then **NTP Client**. **Enabled** is checked, **Mode** is `unicast`, and **NTP Servers** lists `time.google.com` and `pool.ntp.org`.
2. To use different servers, change the list. Pick two or three close to you:

| Region | Server | Notes |
|--------|--------|-------|
| Global | time.google.com | Anycast, works everywhere |
| Global | pool.ntp.org | Round-robin, regional auto-select |
| North America | time.nist.gov | US government time service |
| North America | time.apple.com | Apple's NTP service |
| Europe | europe.pool.ntp.org | European NTP pool |
| Europe | ntp.se | Swedish national time service |
| Asia-Pacific | asia.pool.ntp.org | Asian NTP pool |

> **Tip:** Two servers is enough. If you're traveling, `time.google.com` and `pool.ntp.org` work from anywhere.

3. Click **Apply**. The first synchronization can take up to 30 seconds. **Status** changes to `synchronized`.

> **Note:** If the status doesn't sync, check your internet connection and DNS.

---

## Lab 21.2 — Your NTP server

Your L009 is also an NTP server, again from the script.

4. Click **System**, then **NTP Server**. **Enabled**, **Broadcast**, and **Use Local Clock** are checked. Read the settings and don't change them.

   > **Why:** **Use Local Clock** lets your L009 keep serving time if its internet servers can't be reached. In the **Peers** list from Lab 2, that is the row for `127.127.1.0`.

---

## Lab 21.3 — Hand out your L009 as the time server

Devices on your networks can use your L009 for time. That reduces NTP traffic to the internet and gives them a local source.

5. Click **IP**, then **DHCP Server**, then the **Networks** tab.
6. Double-click the `192.168.88.0/24` row, your backdoor network. Set **NTP Servers** to `192.168.88.1`. Click **Apply**, then **OK**.
7. In the Terminal, run:

```
/ip/dhcp-server/network/print detail where address=192.168.88.0/24
```

The entry shows `ntp-server=192.168.88.1`.

8. Unplug your laptop's Ethernet cable and plug it back in, so it asks for a new lease. Devices that honor the DHCP time server option now use your L009 for time.

> **Note:** Many devices ignore this option. Network gear and many Linux systems use it. Laptops and phones usually use a time setting of their own.

To undo it, clear the **NTP Servers** field.

---

## Lab 21 Summary

| Component | Function |
|-----------|----------|
| NTP Client | Sets your router's clock from internet time servers |
| NTP Server | Serves time to local devices |
| DHCP **NTP Servers** | Tells clients to use your router for time |
