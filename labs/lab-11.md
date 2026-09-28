# Lab 11 — User Manager (RADIUS Server)

*Prerequisites: Lab 2 (packages installed), Lab 6 (backup completed)*

User Manager turns your MikroTik into a RADIUS authentication server. For Wi-Fi professionals, this means you can lab enterprise wireless authentication (WPA2/WPA3-Enterprise) at home without running a separate RADIUS server like FreeRADIUS or Windows NPS.

This lab configures User Manager to authenticate wireless clients connecting to an external enterprise AP. The AP sends authentication requests to the MikroTik, which validates credentials and returns accept/reject. This is the same workflow used in production enterprise networks — just with a fraction of the cost and complexity.

> **Tested Configuration:** This lab was developed and tested using an Enterprise AP under multiple scenarios. The RADIUS configuration will work with any enterprise AP that supports WPA2/WPA3-Enterprise with external RADIUS.

---

## Lab 11.1 — Verify User Manager is Installed

User Manager is a separate package. If you installed it during Lab 2 (Packages), verify it's present. If not, install it now.

### Check for User Manager

1. Look in the left menu for **User Manager**.

2. If it's there, skip to Lab 11.2.

3. If it's not there, check **System** → **Packages** — look for "user-manager" in the list.
   - If present but disabled, enable it and reboot.
   - If not present, continue with the installation steps below.

### Install User Manager (if needed)

If User Manager isn't installed, follow the package installation process from Lab 2:

4. Navigate to **System** → **Resources** and note your **Architecture Name** and **Version**.

5. Download the Extra Packages from https://mikrotik.com/download for your architecture and version.

6. Extract the zip and locate `user-manager-[version]-[architecture].npk`

7. Upload the .npk file to **Files** on your router.

8. Reboot: **System** → **Reboot**

9. After reboot, verify **User Manager** appears in the left menu.

---

## Lab 11.2 — Creating Certificates

User Manager requires certificates for EAP authentication. We'll create:
1. A Certificate Authority (CA) — signs other certificates
2. A server certificate — identifies the RADIUS server to clients
3. A client certificate — for EAP-TLS authentication (optional but useful)

> **Critical:** Use `secp384r1` key size and `sha384` hashing. This combination works reliably with Android, iOS, Windows, and macOS. Other combinations cause "user not found" errors on Android due to anonymous identity handling.

### Understanding the Domain/Realm

Throughout this lab, you'll see usernames formatted as `user1@mikrotik.test`. The part after the @ is called a **realm** — it's just an identifier string, not a real internet domain.

**Key points:**
- The realm doesn't need to be registered, resolvable, or exist anywhere in DNS
- It's purely an organizational label for grouping users
- We use `.test` because it's officially reserved for testing and will never conflict with real domains
- **The only rule:** The certificate's Common Name must exactly match the username in User Manager

You could use `@mylab`, `@home.local`, or `@anything.whatever` — what matters is consistency between your certificates and User Manager database.

### Enable CRL Settings

1. Navigate to **System** → **Certificates**

2. Click the **Settings** button (far right of the window, under **Configuration** in the right-hand column).

3. Configure:
   - **CRL Download:** Checked
   - **Use CRL:** Checked
   - **CRL Store:** ram

4. Click **Apply** & **OK**

### Create the Certificate Authority

5. Click **New** and configure on the **General** tab:
   - **Name:** radius-ca
   - **Common Name:** RADIUS-CA
   - **Digest Algorithm:** sha384
   - **Key Size:** secp384r1
   - **Days Valid:** 1825 (5 years)

6. Click the **Key Usage** tab and check:
   - **key cert. sign**
   - **crl sign**

7. Click **Apply**

8. Click **Sign** (right side of window)

9. Verify **Certificate** is set to **radius-ca**

10. Click **Start**
    > **UI Bug:** If the Sign dialog shows "Error in Certificate - Selection expected," use the CLI instead:
    > ```
    > /certificate/sign radius-ca
    > ```
    > Wait for `progress: done`.

11. Wait for progress to show "done", then click **Cancel** to close the signing window.

12. Click **OK** to return to the certificate list

### Create the Server Certificate

13. Click **New** and configure on the **General** tab:
   - **Name:** radius-server
   - **Common Name:** radius.mikrotik.test
   - **Subject Alt. Name:** DNS:radius.mikrotik.test
   - **Digest Algorithm:** sha384
   - **Key Size:** secp384r1
   - **Days Valid:** 825 (just over 2 years — under Apple's certificate limit)

14. Click the **Key Usage** tab and check:
   - **tls server**

15. Click **Apply**

16. Click **Sign**

17. Set **CA** to **radius-ca**

18. Click **Start**
    > **UI Bug:** If the Sign dialog won't accept the selection, use the CLI:
    > ```
    > /certificate/sign radius-server ca=radius-ca
    > ```
    > Wait for `progress: done`.

19. Wait for "done", then click **Cancel** to close the signing window.

20. Click **OK**

### Create a Client Certificate (for EAP-TLS)

21. Click **New** and configure on the **General** tab:
   - **Name:** user1-client
   - **Common Name:** user1@mikrotik.test
   - **Digest Algorithm:** sha384
   - **Key Size:** secp384r1
   - **Days Valid:** 825

22. Click the **Key Usage** tab and check:
   - **tls client**

23. Click **Apply**

24. Click **Sign**

25. Set **CA** to **radius-ca**

26. Click **Start**
    > **UI Bug:** If the Sign dialog won't accept the selection, use the CLI:
    > ```
    > /certificate/sign user1-client ca=radius-ca
    > ```
    > Wait for `progress: done`.

29. Wait for "done", then click **Cancel** to close the signing window.

30. Click the **General** tab to return to the main certificate settings. Scroll to the bottom of the General tab and locate the **Trusted** checkbox (directly above the Trust Store checkboxes).

31. **Important:** Check the **Trusted** checkbox. This marks the certificate as trusted for client authentication.

32. Click **Apply** & **OK**

### Verify Certificates

You should now have three new RADIUS certificates (in addition to any pre-existing certificates on your device):

| Name | Flags | Type | Key Usage |
|------|-------|------|-----------|
| radius-ca | **KAT** | CA (self-signed) | key cert. sign, crl sign |
| radius-server | **KI** | Server (issued by radius-ca) | tls server |
| user1-client | **KIT** | Client (issued by radius-ca) | tls client |

**Understanding the flags:**
- **K** = Private Key present
- **A** = Authority (this is a CA)
- **I** = Issued (signed by a CA)
- **T** = Trusted

> **Note:** Only the CA certificate shows the **A** (Authority) flag. The server and client certificates show **I** (Issued) because they were signed by the CA. The client certificate shows **T** (Trusted) because you checked the Trusted checkbox in step 31.

---

## Lab 11.3 — Configuring User Manager

### Enable User Manager

1. Navigate to **User Manager** in the left menu.

2. Click **Settings** (far right of the window, under **Configuration** in the right-hand column).

3. Configure:
   - **Enabled:** Checked
   - **Certificate:** radius-server

4. Click **Apply** & **OK**

### Add RADIUS Client (Your Enterprise AP)

User Manager needs to know which devices are allowed to send RADIUS requests. Each AP (or WLC) that will authenticate against User Manager needs an entry.

**For WLPC classes, use the values in the brackets below.**

5. In the **Routers** tab, click **New**

6. Configure:
   - **Name:** *make-model* (use a descriptive name like "ruckus-r770" or "aruba-ap22") [mikrotik-ap]
   - **Address:** (IP address of your AP on the management VLAN) [10.22.255.0/24]
   - **Shared Secret:** [create a strong shared secret — you'll need this when configuring the AP]

   > **Example:** If your AP will get 10.10.255.x from DHCP, use that address. For testing, you can use 10.10.255.0/24 to allow any device on that subnet, but specific IPs are more secure.

7. Click **Apply** & **OK**

### Configure Authentication Methods

8. Click the **User Groups** tab (not the Users tab — these are different).

9. Click on the **default** group to edit it.

10. In the **Outer Auths** section:
    - **Uncheck** everything except:
      - **EAP-PEAP** (username/password)

    > **Note:** Leave the **Inner Auths** checkboxes as they are. For EAP-PEAP, MSCHAPv2 needs to remain checked as the inner authentication method. For EAP-TLS, inner auths are not used since the certificate handles authentication directly.

    > > **Note:** EAP-TLS is handled by the dedicated `cert-auth` group we'll create next. Keeping auth methods separated by group gives you cleaner control over who authenticates how.

11. Click **Apply** & **OK**

12. Click **New** to create a certificate-only group:
    - **Name:** cert-auth
    - **Outer Auths:** Check only **EAP-TLS**
    - **Inner Auths:** Uncheck everything

13. Click **Apply** & **OK**

### Add Test Users

14. Click the **Users** tab.

15. Click **New** to create an EAP-TLS user:
    - **Name:** user1@mikrotik.test (must match the client certificate CN)
    - **Group:** cert-auth
    - **Comment:** EAP-TLS test user

16. Click **Apply** & **OK**

17. Click **Add New** to create an EAP-PEAP user:
    - **Name:** user2@mikrotik.test
    - **Password:** [create a password and record it]
    - **Group:** default
    - **Shared Users:** 3 (allows 3 simultaneous connections)
    - **Comment:** EAP-PEAP test user

18. Click **Apply** & **OK**

---

## Lab 11.4 — Firewall Rules for RADIUS

The AP needs to reach User Manager on UDP ports 1812 (authentication) and 1813 (accounting).

1. Navigate to **IP** → **Firewall**

2. Click **New** and configure on the **General** tab:
   - **Comment:** `RADIUS from AP`
   - **Chain:** input
   - **Protocol:** udp
   - **Dst. Port:** 1812,1813
   - **In. Interface:** vlan255 (or wherever your AP connects)

3. Click the **Action** tab:
   - **Action:** accept

4. Click **Apply** & **OK**

5. Drag the rule up so it's processed before any drop rules.

---
> **Class note:** Labs 11.5 and 11.6 require a configured AP, which we haven't built yet. Stop here — User Manager is configured and ready. We'll come back to test RADIUS authentication after Lab 13 when your mAP is set up as an AP.
---

## Lab 11.5 — Configure Your Enterprise AP

This section provides general guidance. Specific steps vary by vendor.

### Required Information

| Item | Value |
|------|-------|
| RADIUS Server IP | 10.10.255.1 (your MikroTik's VLAN 255 address) |
| RADIUS Port | 1812 |
| RADIUS Secret | [the shared secret from Lab 11.3 step 6] |
| Accounting Port | 1813 (optional) |

### RUCKUS Unleashed Example

1. Log into your Unleashed AP's web interface.

2. Navigate to **Admin & Services** → **Services** → **AAA Servers**

3. Click **Create New**

4. Configure:
   - **Name:** MikroTik-RADIUS
   - **Type:** RADIUS
   - **Auth Server IP:** 10.10.255.1
   - **Auth Port:** 1812
   - **Auth Secret:** [your shared secret]

5. Click **OK**

6. Navigate to **Wi-Fi Networks** and edit (or create) your WPA2-Enterprise SSID.

7. Under **Security**:
   - **Authentication:** WPA2
   - **Encryption:** AES
   - **Authentication Server:** MikroTik-RADIUS

8. Save the configuration.

### Other Vendors

**Cisco:**
```
radius server MIKROTIK
  address ipv4 10.10.255.1 auth-port 1812 acct-port 1813
  key [shared-secret]
```

**Aruba:**
```
aaa authentication-server radius "MikroTik"
  host 10.10.255.1
  key [shared-secret]
```

---

## Lab 11.6 — Testing Authentication

### Test EAP-PEAP (Username/Password)

1. On a test device (phone or laptop), connect to your WPA2-Enterprise SSID.

2. When prompted:
   - **EAP Method:** PEAP
   - **Phase 2 Authentication:** MSCHAPv2
   - **Identity:** user2@mikrotik.test
   - **Password:** [the password you created in Lab 11.3]
   - **CA Certificate:** Do not validate (for lab testing) or install the CA cert

3. The device should authenticate and receive an IP address.

### Test EAP-TLS (Certificate)

For EAP-TLS, you need to export and install the client certificate on your test device. The MikroTik app makes this much easier than manual file transfer.

**Using the MikroTik App (Recommended for phones/tablets):**

1. Install the MikroTik app on your phone (available for iOS and Android).

2. Connect to your router through the app.

3. Navigate to **System** → **Certificates**

4. Select the client certificate (user1-client)

5. Export the certificate — the app handles the transfer and installation directly to your device's certificate store.

6. Connect to the WPA2-Enterprise SSID and select the installed certificate.

**Manual Export (for laptops or devices without the app):**

1. Navigate to **System** → **Certificates**

2. Select the client certificate (user1-client)

3. Click **Export**

4. Configure:
   - **Type:** PKCS12
   - **Export Passphrase:** [create a passphrase]

5. Click **Export**

6. Navigate to **Files** and download the .p12 file.

7. Transfer the .p12 file to your device and install it.

8. Connect to the WPA2-Enterprise SSID using the certificate.

### Verify in User Manager

1. Navigate to **User Manager** → **Sessions**

2. You should see active sessions for authenticated users.

3. Navigate to **User Manager** → **Users** and click on a user to see their session history.

---

## Lab 11.7 — Troubleshooting RADIUS

### Watch RADIUS Traffic with Torch

Torch is MikroTik's real-time traffic monitoring tool. It's covered in detail in Lab 21, but here's a quick look at using it for RADIUS troubleshooting.

1. Navigate to **Tools** → **Torch**

2. Configure:
   - **Interface:** vlan255 (or your AP-facing interface)
   - **Src. Address:** [your AP's IP]
   - **Protocol:** udp
   - **Port:** 1812

3. Click **Start**

4. Attempt authentication from a client.

5. You should see UDP traffic on port 1812 between the AP and MikroTik.

### Check User Manager Logs

1. Navigate to **Log**

2. Look for entries containing "radius" or "user-manager"

3. Common errors:
   - "user not found" — username doesn't match User Manager database
   - "shared secret mismatch" — secret on AP doesn't match MikroTik
   - "certificate error" — certificate chain issues

### Common Issues

| Symptom | Cause | Solution |
|---------|-------|----------|
| No RADIUS traffic | Firewall blocking | Check firewall rules for UDP 1812/1813 |
| "User not found" | Username mismatch | Verify username matches exactly (including @domain) |
| Android fails, others work | Key size issue | Ensure certificates use secp384r1 |
| "Invalid certificate" | CA not trusted | Export and install CA certificate on client |
| Timeout | Wrong IP/port | Verify AP is configured with correct RADIUS server IP |

---

## Lab 11 Summary

You now have:
- ✅ User Manager installed and configured as a RADIUS server
- ✅ Certificate infrastructure for EAP-TLS
- ✅ Test users for both EAP-PEAP and EAP-TLS
- ✅ Firewall rules allowing RADIUS traffic
- ✅ Enterprise AP configured to authenticate against MikroTik

This configuration lets you lab WPA2/WPA3-Enterprise authentication at home using real enterprise APs — no separate RADIUS server required.

> **Next Steps:** Lab 19 covers connecting an enterprise switch and AP to your MikroTik using trunk ports, which provides the physical infrastructure for testing this RADIUS configuration.

---
