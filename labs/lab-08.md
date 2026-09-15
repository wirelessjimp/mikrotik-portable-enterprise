# Lab 8 — Setting Up DHCP

*Prerequisites: Lab 7*

Without DHCP, devices can't automatically get IP addresses. This lab builds DHCP infrastructure for each VLAN.

---

## Lab 8.1 — Setting Up IP Addresses

First, assign IP addresses to each VLAN interface. These become the default gateways for each VLAN.

### Create IP Addresses

1. Navigate to **IP** → **Addresses**

2. Click **Add New** and configure:
   - **Comment:** VLAN 20
   - **Address:** 10.10.20.1/24
   - **Network:** 10.10.20.0 (click the **+** button to expand)
   - **Interface:** vlan20
   - **Enabled:** Checked

   > **Important:** Assign the IP address to the **VLAN interface** (vlan20), not the bridge (vlan20bridge). When VLAN filtering is enabled on the bridge, traffic arrives on the VLAN interface. Services like DHCP will not function correctly if the IP address is on the bridge instead of the VLAN interface.

3. Click **Apply** and **OK**

4. Create the second IP address using the UI:
   - **Comment:** VLAN 30
   - **Address:** 10.10.30.1/24
   - **Network:** 10.10.30.0
   - **Interface:** vlan30

5. Create the remaining IP addresses using CLI:

    ```
    /ip/address/add address=10.10.40.1/24 network=10.10.40.0 interface=vlan40 comment="VLAN 40"
    /ip/address/add address=10.10.255.1/24 network=10.10.255.0 interface=vlan255 comment="VLAN 255"
    ```

When complete, you should have:

| Comment | Address | Network | Interface |
|---------|---------|---------|-----------|
| VLAN 20 | 10.10.20.1/24 | 10.10.20.0 | vlan20 |
| VLAN 30 | 10.10.30.1/24 | 10.10.30.0 | vlan30 |
| VLAN 40 | 10.10.40.1/24 | 10.10.40.0 | vlan40 |
| VLAN 255 | 10.10.255.1/24 | 10.10.255.0 | vlan255 |

> **IP Addressing Pattern:** The third octet matches the VLAN ID. This makes troubleshooting easier — if a device has IP 10.10.30.x, you immediately know it's on VLAN 30.

---

## Lab 8.2 — Setting Up IP Pools

IP pools define which addresses DHCP can hand out.

### Create Address Pools

1. Navigate to **IP** → **Pool**

2. Click **Add New** and configure:
   - **Comment:** VLAN 20
   - **Name:** vlan20
   - **Addresses:** 10.10.20.10-10.10.20.250
   - **Next Pool:** none

3. Click **Apply** and **OK**

4. Create the second pool using the UI:
   - **Comment:** VLAN 30
   - **Name:** vlan30
   - **Addresses:** 10.10.30.10-10.10.30.250
   - **Next Pool:** none

5. Create the remaining pools using CLI:

    ```
    /ip/pool/add name=vlan40 ranges=10.10.40.10-10.10.40.250 comment="VLAN 40"
    /ip/pool/add name=vlan255 ranges=10.10.255.10-10.10.255.250 comment="VLAN 255"
    ```

> **Best Practice:** Don't use the full range. Reserve .1-.9 for static assignments (servers, printers, etc.) and .251-.254 for network infrastructure.

> **Syntax Note:** No spaces in the address range. `10.10.20.10-10.10.20.250` works; `10.10.20.10 - 10.10.20.250` fails.

---

## Lab 8.3 — Setting Up DHCP Servers

### Create DHCP Servers

1. Navigate to **IP** → **DHCP Server**

2. Click **Add New** and configure:
   - **Name:** vlan20
   - **Interface:** vlan20
   - **Lease Time:** 02:00:00 (2 hours)
   - **Address Pool:** vlan20
   - **Add ARP For Leases:** Checked (scroll down to find this)
   - **Enabled:** Checked

   > **Important:** Bind the DHCP server to the **VLAN interface** (vlan20), not the bridge (vlan20bridge). This must match where the IP address was assigned in Lab 8.1. If the DHCP server is bound to an interface without an IP address, it will show as INVALID and will not issue leases.

3. Click **Apply** and **OK**

4. Create the second DHCP server using the UI:
   - **Name:** vlan30
   - **Interface:** vlan30
   - **Lease Time:** 02:00:00
   - **Address Pool:** vlan30
   - **Add ARP For Leases:** Checked

5. Create the remaining DHCP servers using CLI:

    ```
    /ip/dhcp-server/add name=vlan40 interface=vlan40 lease-time=02:00:00 address-pool=vlan40 add-arp=yes
    /ip/dhcp-server/add name=vlan255 interface=vlan255 lease-time=02:00:00 address-pool=vlan255 add-arp=yes
    ```

### Configure DHCP Networks

> **⚠ Critical: Do not skip this section.** Creating DHCP servers (above) is not enough — each server also requires a network definition in the **Networks** tab. Without it, the DHCP server is bound to the interface but has no network parameters to hand out. Clients will send DHCP discover requests and receive nothing in return. This is a silent failure — the server appears configured but is completely non-functional. Complete every network entry below before testing.

6. Click the **Networks** tab at the top.

7. Click **Add New** and configure:
   - **Comment:** VLAN 20
   - **Address:** 10.10.20.0/24
   - **Gateway:** 10.10.20.1 (click the **+** button to expand)
   - **Netmask:** 255.255.255.0
   - **DNS Servers:** 10.10.20.1

8. Click **Apply** and **OK**

9. Create the second network using the UI:
   - **Comment:** VLAN 30
   - **Address:** 10.10.30.0/24
   - **Gateway:** 10.10.30.1
   - **Netmask:** 255.255.255.0
   - **DNS Servers:** 10.10.30.1

10. Create the remaining networks using CLI:

    ```
    /ip/dhcp-server/network/add address=10.10.40.0/24 gateway=10.10.40.1 netmask=255.255.255.0 dns-server=10.10.40.1 comment="VLAN 40"
    /ip/dhcp-server/network/add address=10.10.255.0/24 gateway=10.10.255.1 netmask=255.255.255.0 dns-server=10.10.255.1 comment="VLAN 255"
    ```

---

## Lab 8.9 — DHCP Option 43 (Future Reference)

*Reference: Use when deploying access points that need to find a local controller.*

Most AP vendors use DHCP Option 43 to tell APs where to find their controller. This section documents the process for future deployments.

### Configuration Steps

1. Navigate to **IP** → **DHCP Server** → **Options** tab

2. Click **Add New**:
   - **Name:** (Vendor) Option 43 — use the vendor name to identify the controller
   - **Code:** 43
   - **Value:** `0x[hex_code]` — see vendor codes below

3. Click **Apply** and **OK**

4. Navigate to **Option Sets** tab

5. Click **Add New**:
   - **Name:** (Vendor) Option 43
   - **Options:** Select the option you created in step 2

6. Click **Apply** and **OK**

7. Navigate to **DHCP** tab

8. Click on the DHCP server for the subnet where APs will connect

9. Find **DHCP Option Set** and select your option set

10. Click **Apply** and **OK**

### Understanding the Hex Value

The value format is: `0x` + `Vendor Code` + `Length` + `IP Address in Hex`

For an example controller IP of **10.10.10.20**:

| Vendor | Value |
|--------|-------|
| RUCKUS SZ | `0x060b31302e31302e31302e3230` |
| Cisco | `0xf1040a0a0a14` |
| Extreme | `0xe2040a0a0a14` |
| Ubiquiti | `0x01040a0a0a14` |
| Fortinet | `0x2b1a0a2e0a2e0a2e142e` |
| Aruba | Requires Option 60 set to "ArubaAP" + Option 43: `0x0a0a0a14` |

### Helpful Calculators

- https://wifiwizardofoz.com/dhcp-option-43-calculator/
- https://shimi.net/services/opt43/

---
