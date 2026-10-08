# Lab 10 — MikroTik Cloud

*Prerequisites: Lab 1 (WAN connected)*

**Why:** MikroTik Cloud gives your router a name on the internet that follows its address when the address changes (DDNS), and it can keep the router's clock right. Back to Home (Lab 11) needs it. The instructor's script already turned it on, so in this lab you look and record, not build.

1. Click **IP**, then **Cloud**. The window opens on the **Cloud** tab.
2. Check these, in the order the window lists them:
   - **DDNS Enabled:** `auto`
   - **DDNS Update Interval:** `01:00:00`
   - **Update Time:** checked
   - **Public Address:** your router's address as the internet sees it
   - **DNS Name:** your router's name, ending in `.sn.mynetname.net`
3. At the bottom left, the status reads `updated`. If it shows an error, click **Force Update** under **Actions**.

   > **Note:** At the bottom right, the window says `Router is behind a NAT. Remote connection might not work.` That's expected here, because the class router sits between your kit and the internet. Back to Home is built to work through that.

4. Record the **DNS Name** in **Lab Notes**, in the **DDNS Address** row.

   > **Why:** Each router gets its own name. It's the address you'd use to reach this router from anywhere, once the router is allowed to answer.

5. Leave **Update Time** checked. It sets the router's clock from MikroTik's servers, which matters for certificates and log times.
