# Lab 19 — Enterprise AP Integration

*Prerequisites: Lab 11 (User Manager/RADIUS), Lab 18 (switch configured with AP port)*

This lab connects an enterprise AP to your network, configures it to use your MikroTik as a RADIUS server, and validates WPA2/WPA3-Enterprise authentication.

> **Tested Configuration:** This lab was developed using a RUCKUS AP running Unleashed firmware. The RADIUS and VLAN concepts apply to any enterprise AP — adjust the AP configuration steps for your vendor's interface.

---

## Lab 19.1 — Connect the AP

### Physical Connection

1. Connect your AP to the switch port configured in Lab 18.7 (or directly to your MikroTik's expansion port if not using a switch).

2. Power the AP:
   - Via PoE from the switch
   - Via PoE injector
   - Via power adapter (if applicable)

3. Wait for the AP to boot. Watch status LEDs — most APs indicate ready state with a solid or slowly blinking LED.

### Verify AP Gets IP Address

4. On your MikroTik, navigate to **IP** → **DHCP Server** → **Leases**

5. Look for a new lease on the vlan255bridge DHCP server — this is your AP.

6. Record the AP's IP address:

   > **AP Management IP:** ________________________________

---

## Lab 19.2 — Initial AP Configuration

Access your AP's management interface to perform initial setup. Steps vary by vendor.

### Access AP Web Interface

1. Open a browser and navigate to **https://[AP IP Address]**

2. Accept any certificate warnings (APs typically use self-signed certificates).

3. Log in with default credentials (check your AP's documentation — often printed on the AP label).

### Basic Configuration

4. Set the AP's hostname/system name.

5. Change the default admin password.

6. Set the country/regulatory domain.

7. Verify the AP has internet connectivity (for firmware updates if needed).

8. Update firmware if a newer version is available.

9. Save the configuration.

> **Note:** We're not covering vendor-specific setup wizards here. Complete your AP's initial configuration per its documentation before proceeding.

---

## Lab 19.3 — Configure RADIUS Authentication

Now we point the AP to your MikroTik's User Manager for 802.1X authentication.

### Information Needed

| Item | Value |
|------|-------|
| RADIUS Server IP | 10.10.255.1 |
| RADIUS Auth Port | 1812 |
| RADIUS Acct Port | 1813 |
| RADIUS Shared Secret | [From Lab 11.3] |

### Add RADIUS Server on AP

1. In your AP's management interface, find the RADIUS or AAA server configuration.
   - Often under: Security, Authentication, AAA, or Services

2. Add a new RADIUS server:
   - **Name/Description:** MikroTik-RADIUS (or similar)
   - **Type:** RADIUS (or Authentication Server)
   - **IP Address:** 10.10.255.1
   - **Port:** 1812
   - **Shared Secret:** [Your RADIUS shared secret from Lab 11]

3. Save the configuration.

4. If your AP supports RADIUS accounting, add a second entry:
   - **IP Address:** 10.10.255.1
   - **Port:** 1813
   - **Shared Secret:** [Same secret]

### Test RADIUS Connectivity

Many APs have a "Test" button for RADIUS servers. If available:

1. Enter a test username: `user2@mikrotik.test`
2. Enter the password from Lab 11.3
3. Run the test

If the test succeeds, RADIUS is working. If not, check:
- Can the AP ping 10.10.255.1?
- Is UDP 1812/1813 allowed through the firewall (Lab 11.4)?
- Does the shared secret match exactly?

---

## Lab 19.4 — Create WPA2-Enterprise SSID

Create an SSID that uses RADIUS authentication.

### SSID Configuration

1. Navigate to your AP's wireless/WLAN/SSID configuration.

2. Create a new SSID (or edit an existing one):
   - **SSID Name:** Lab-Enterprise
   - **Security Mode:** WPA2-Enterprise (or WPA2/WPA3-Enterprise)
   - **Encryption:** AES/CCMP
   - **Authentication Server:** MikroTik-RADIUS (the server you added)

3. Configure VLAN assignment:
   - **VLAN:** 20 (or your preferred client VLAN)
   
   This means authenticated clients land on VLAN 20, not the management VLAN.

4. Save and apply the configuration.

### Optional: Additional Wireless Settings

Depending on your AP, consider:
- **Band:** 5 GHz preferred, or both bands
- **Minimum data rate:** 24 Mbps (disables low 802.11b/g rates)
- **802.11k/v/r:** Enable if supported (improves roaming)

---

## Lab 19.5 — Test EAP-PEAP Authentication

Test with username/password authentication.

### Connect a Client

1. On a test device (laptop, phone, tablet), find the **Lab-Enterprise** SSID.

2. Connect. When prompted for credentials:
   - **EAP Method:** PEAP
   - **Phase 2 Authentication:** MSCHAPv2
   - **Identity/Username:** user2@mikrotik.test
   - **Password:** [From Lab 11.3]
   - **CA Certificate:** Do not validate (or install your CA cert for production)

3. The device should authenticate and connect.

### Verify on Client Device

4. Check the client's IP address — it should be in **10.10.20.0/24** (VLAN 20).

5. Test connectivity:
   - Ping 10.10.20.1 (MikroTik gateway for VLAN 20) ✓
   - Ping 8.8.8.8 (internet) ✓

### Verify on MikroTik

6. Navigate to **User Manager** → **Sessions**

7. You should see an active session for `user2@mikrotik.test`.

8. Navigate to **IP** → **DHCP Server** → **Leases**

9. Find your client — verify it shows the vlan20bridge DHCP server.

---

## Lab 19.6 — Test EAP-TLS Authentication (Optional)

Test with certificate-based authentication. This requires the client certificate from Lab 11.6.

### Install Client Certificate

1. Ensure you've exported and installed the client certificate (.p12 file) on your test device.

2. The certificate must be trusted by the device.

### Connect a Client

3. On your test device, connect to **Lab-Enterprise**.

4. When prompted:
   - **EAP Method:** TLS
   - **Identity/Username:** user1@mikrotik.test
   - **Client Certificate:** [Select the installed certificate]
   - **CA Certificate:** [Your RADIUS CA, or "Do not validate" for lab]

5. The device should authenticate using the certificate (no password needed).

### Verify Authentication

6. Check the client IP — should be in 10.10.20.0/24.

7. Check User Manager Sessions — should show `user1@mikrotik.test`.

---

## Lab 19.7 — Verify VLAN Assignment

Confirm clients land on the correct VLAN based on SSID configuration.

### Check Client Placement

1. Connect a device to **Lab-Enterprise**.

2. Verify IP is in 10.10.20.0/24.

3. If you created multiple SSIDs with different VLANs:
   - SSID on VLAN 20 → Client gets 10.10.20.x
   - SSID on VLAN 30 → Client gets 10.10.30.x

### Test Isolation

4. Connect two devices:
   - Device A on VLAN 20 (via Lab-Enterprise)
   - Device B on VLAN 255 (via wired or different SSID)

5. Try to ping Device B from Device A.

6. This should **fail** (firewall rules from Lab 10 block cross-VLAN traffic).

---

## Lab 19.8 — Troubleshooting

### AP Can't Reach RADIUS Server

**Symptoms:** RADIUS test fails, clients can't authenticate

**Check:**
1. AP has IP on VLAN 255? (Check DHCP leases)
2. AP can ping 10.10.255.1? (Test from AP CLI if available)
3. Firewall allows UDP 1812/1813 from AP? (Lab 11.4)
4. Shared secret matches exactly? (Case-sensitive, no extra spaces)

### Client Authentication Fails

**Symptoms:** Client prompts for credentials but never connects

**Check MikroTik Log:**
1. Navigate to **Log** in WinBox
2. Look for User Manager entries
3. Common messages:
   - "user not found" — Username doesn't match User Manager
   - "shared secret mismatch" — Secret doesn't match
   - "certificate error" — Certificate issue (check key type)

**Use Torch to verify traffic:**
1. Navigate to **Tools** → **Torch**
2. Set **Src. Address:** [AP's IP]
3. Set **Protocol:** UDP
4. Set **Port:** 1812
5. Start — you should see RADIUS requests from the AP

### Client Gets Wrong IP/VLAN

**Symptoms:** Client connects but gets IP from wrong VLAN

**Check:**
1. SSID is configured with correct Access VLAN on AP?
2. Switch port tags that VLAN to the AP?
3. MikroTik has DHCP server for that VLAN?
4. VLAN interface exists on trunk port?

### Client Can't Reach Internet

**Symptoms:** Client authenticates, gets IP, but no internet

**Check:**
1. Can client ping gateway (10.10.20.1)?
2. Can client ping 8.8.8.8?
3. Firewall rules allow traffic from VLAN 20 to internet?
4. NAT/masquerade rule includes VLAN 20?

---

## Lab 19 Summary

You now have:

- ✅ Enterprise AP connected via switch (or direct trunk)
- ✅ RADIUS server configured on AP pointing to MikroTik
- ✅ WPA2/WPA3-Enterprise SSID broadcasting
- ✅ EAP-PEAP authentication tested (username/password)
- ✅ EAP-TLS authentication tested (certificate) — optional
- ✅ Clients landing on correct VLANs
- ✅ Full enterprise wireless lab environment

**What you've built:**

This is a complete enterprise wireless lab:
- MikroTik as router, DHCP server, and RADIUS server
- Enterprise switch extending VLANs with PoE
- Enterprise AP with WPA2/WPA3-Enterprise
- Multiple VLANs for client segmentation

You can now practice:
- WPA2/WPA3-Enterprise authentication
- EAP-PEAP and EAP-TLS methods
- VLAN assignment per SSID
- RADIUS troubleshooting
- Enterprise AP configuration

All with hardware that fits in a small bag and costs less than a single enterprise controller license.

---

## Lab Notes — Lab 19

| Item | Value |
|------|-------|
| AP Make/Model | |
| AP Management IP | |
| AP Admin Password | |
| RADIUS Shared Secret | |
| Enterprise SSID Name | |
| Enterprise SSID VLAN | |

**Authentication Test Results:**

| Test | Result |
|------|--------|
| RADIUS connectivity test | ☐ Pass ☐ Fail |
| EAP-PEAP authentication | ☐ Pass ☐ Fail |
| EAP-TLS authentication | ☐ Pass ☐ Fail ☐ Skipped |
| Client gets correct VLAN IP | ☐ Pass ☐ Fail |
| Cross-VLAN traffic blocked | ☐ Pass ☐ Fail |
| Client can reach internet | ☐ Pass ☐ Fail |

---

*Document Version: Draft 1.0*
*Last Updated: March 2026*
