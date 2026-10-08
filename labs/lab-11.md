# Lab 11 — Back to Home

*Prerequisites: Lab 10 (MikroTik Cloud). The MikroTik Back To Home app is installed on your phone before class.*

**Why:** Back to Home is MikroTik's simple VPN. It uses WireGuard underneath, but it handles the keys for you, and it connects through MikroTik's relay servers, so nothing has to be forwarded through the routers in between. The instructor's script already turned it on, so you check it and connect a phone.

1. **L009 window:** click **IP**, then **Cloud**, then the **BTH VPN** tab. Check that **Back To Home VPN** is `enabled`, **VPN Status** is `running`, and **VPN DNS Name** ends in `.vpn.mynetname.net`. **VPN Relay IPv4 Status** reads `reachable via relay` for at least one relay.

   > **Note:** The bottom of the window may say `Router is behind a NAT. Remote connection might not work.` That warning is about reaching your router directly. Back to Home doesn't do that. Both your router and your phone connect out to the relay.

2. Click the **BTH VPN WireGuard** tab. It shows a client configuration and a QR code.
3. On your phone, open the **MikroTik Back To Home** app. It asks for a few basic permissions. Allow them.
4. In the app, tap **Join shared**, then **Scan QR code**. Point the phone at the QR code on your L009's screen.
5. **iPhone:** after the scan, the app offers a name for the connection. Change it to something you'll recognize, or leave it. Then follow the prompts, including the iPhone's own message about adding a VPN.

   **Android:** the app asks to add a VPN, with no chance to name the connection. Allow it.

6. Tap **Connect** in the app. The prompts on each phone explain what to tap.
7. **L009 window:** click **WireGuard**, then the **Peers** tab, and scroll right. The `peer1` row on `back-to-home-vpn` shows a **Current Endpoint** that's one of the relay addresses from the **BTH VPN** tab, a recent **Last Handshake**, and traffic in both directions.

   > **Why:** That row is your phone's tunnel, seen from the router. The endpoint is the relay, not your phone, which is why the tunnel works without any port forwarding.
8. Now use it from outside. Put your phone on a different network from your kit's: the hotel Wi-Fi, or cellular data with Wi-Fi turned off. Check for roaming charges before you use cellular. Tap **Connect** in the Back To Home app. Open the **MikroTik** app and log in to `10.10.255.1` as `admin` with your L009 password. The first screen shows your L009 (`Student31`). Open a graph of the `ether1` traffic.

   > **Why:** `10.10.255.1` is your router's own address on its management network. The tunnel carries your phone to it, from anywhere in the world.

> ### ⚠️ STOP AND READ
> The QR code and the configuration under it include the key that lets a device into your network. Don't photograph it or share it, and close the window when you're done.
