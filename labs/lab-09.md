# Lab 9 — Testing Port Configurations

*Prerequisites: Lab 7, Lab 8*

After configuring bridges, VLANs, and DHCP, verify everything works.

---

## Lab 9.1 — Test VLAN Connectivity

### Test Each Port

1. Ensure your laptop's wired NIC is set to **DHCP**

2. Connect to **ether2** (VLAN 20)

3. Wait for an IP address. You should receive an address in **10.10.20.0/24**

4. If you receive an APIPA address (169.254.x.x), something is misconfigured. Check:
   - Is ether2 in vlan20bridge?
   - Does the **vlan20** interface have an IP address (10.10.20.1/24)?
   - Is the DHCP server running, enabled, and bound to **vlan20** (not vlan20bridge)?
   - Is the DHCP network configured with correct gateway?
   - Is hardware offload disabled on ether2 in Bridge → Ports?
   - Is ether2 in the LAN interface list?

5. Move the cable to **ether3** (VLAN 30)

6. You should receive an address in **10.10.30.0/24**

7. Repeat for each configured port

### Verify Pattern

With this configuration, the third octet identifies the VLAN:
- 10.10.**20**.x = VLAN 20 (ether2)
- 10.10.**30**.x = VLAN 30 (ether3)
- 10.10.**40**.x = VLAN 40 (ether4)
- 10.10.**255**.x = VLAN 255 (expansion port)

This makes troubleshooting faster — you can identify the VLAN from any IP address at a glance.

---

## Lab 9.2 — Test Container Access

While connected to a VLAN port, verify you can reach the containers we built in Lab 5.

1. Stay connected to a VLAN port (e.g., ether2 for VLAN 20)

2. Open a browser and navigate to: **http://172.17.0.2:3000**

3. You should see the OpenSpeedTest interface.

4. Try running a speed test to verify full connectivity.

### Why This Works

The containers live on 172.17.0.x (the dockers bridge). Your VLAN device is on 10.10.20.x. Traffic flows because:

1. The MikroTik is the gateway for both networks
2. The firewall rule allowing traffic to `!10.0.0.0/8` (non-10.x destinations) permits traffic to 172.17.0.x
3. No additional configuration needed — routing just works

> **Note:** If you later add more restrictive firewall rules (like a "drop all else" rule), you may need to add explicit accept rules for container access. See Lab 10.9 for details.

---

## Lab 9.3 — Check DHCP Leases

1. Disconnect from the MikroTik's LAN ports

2. Connect to your home network (WAN side) or use the backdoor port

3. Access your router via its WAN IP or https://192.168.88.1

4. Navigate to **IP** → **DHCP Server** → **Leases** (third tab)

5. You should see entries for each IP address your laptop received during testing

---
