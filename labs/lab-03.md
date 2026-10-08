# Lab 3 — Certificates and HTTPS

*Prerequisites: Lab 1 (WAN connected, time correct), Lab 2*

**Why:** Many services you'll use later, including the router's web interface and RADIUS, depend on certificates. You'll build your own certificate authority (CA), use it to sign two certificates, and turn on HTTPS for the router. *(First draft of this paragraph.)*

### 3.1 Create the CA

1. Click **System** in the left menu, then **Certificates**. The list is empty.
2. Click **New**. On the **General** tab, set:
   - **Name:** `local-ca`
   - **Common Name:** `local-ca`
   - **Digest Algorithm:** click the **+** and choose `sha384`
   - **Key Size:** `secp384r1`
   - **Days Valid:** `3650`

   > **Note:** Leave **Country**, **State**, **Locality**, **Organization**, **Unit**, and **Subject Alt. Name** empty. Leave **Key Type** alone. It shows `RSA` now and changes to `EC` when you click **Apply**. That's expected, because `secp384r1` is an elliptic curve.

3. Click the **Key Usage** tab. All the boxes start empty. Check these two:
   - **crl sign** (left column, fourth row)
   - **key cert. sign** (right column, third row)

   > **Note:** The blue outline around **digital signature** is just the cursor, not a checkmark.

4. Click **Apply**. The window title changes from **New...** to **local-ca**, and the **Apply** button turns gray.
5. In the **Actions** list on the right, click **Sign**.
6. In the **Sign** dialog, check that **Certificate** says `local-ca`. Leave **CA** and **CA CRL Host** alone, then click **Start** **once**.

   > **Why:** A CA has no one above it to sign it, so it signs itself. Leaving **CA** empty tells the router that.

   > **Note:** Clicking **Start** a second time makes the router try to sign it again, and that fails. If you see an error after you double-click, the first click probably worked. Check the certificate list.

7. Wait for **Progress** to show `done`, then close the **Sign** dialog and click **OK**. The certificate list shows `local-ca` with the flags **KAT**.

### 3.2 Create and sign the web certificate

8. Click **New**. On the **General** tab, set:
   - **Name:** `ssl-web-config`
   - **Common Name:** `ssl-web-config`
   - **Days Valid:** `730`

   > **Note:** Leave **Digest Algorithm**, **Key Type**, and **Key Size** alone. This certificate stays `RSA` with a `2048` key, unlike `local-ca`.

9. Click the **Key Usage** tab and check **digital signature**, **key encipherment**, and **tls server**.
10. Click **Apply**. The **Apply** button turns gray, and the certificate appears in the list with no flags.
11. In the certificate window, click **Sign** in the **Actions** list. Set **CA** to `local-ca` and click **Start**. The dialog shows `Error in Certificate - Selection expected!`

    > ### ⚠️ STOP AND READ
    > The Sign dialog sometimes fails with this error. It isn't your mistake. Don't keep clicking **Start**. Click **Cancel** and sign it from the Terminal instead.

12. Click **OK** to close the certificate window. The certificate stays in the list, unsigned and with no flags.
13. Click **New Terminal** in the left menu, type this, and press **Enter**:

```
/certificate/sign ssl-web-config ca=local-ca
```

14. The Terminal prints `progress: done`. In the certificate list, `ssl-web-config` now shows the flags **KI**.

### 3.3 Create and sign the RADIUS server certificate

15. Click **New**. On the **General** tab, set these in the order the window lists them:
    - **Name:** `radius-server`
    - **Common Name:** `radius.mikrotik.test`
    - **Subject Alt. Name:** click the **+**, choose **DNS**, click in the box after the colon, **delete the `::` that's already in it**, and type `radius.mikrotik.test`
    - **Digest Algorithm:** click the **+** and choose `sha384`
    - **Key Size:** `secp384r1`
    - **Days Valid:** `825`

    > ### ⚠️ STOP AND READ
    > Clear the `::` before you type. If you don't, the name becomes `::radius.mikrotik.test`, and the certificate will be wrong.

    > **Why:** The **Common Name** is the certificate's label. The **Subject Alt. Name** is the list of names a device actually checks when it connects to the server, and many clients ignore the Common Name and look only at this list. Setting both to the same name keeps them from disagreeing. The name doesn't have to exist in DNS. It only has to match what you enter in the RADIUS settings later. *(Describes how TLS clients check names in general. Not tested against Android, iOS, or the class laptops.)*

    > **Note:** **Key Type** changes from `RSA` to `EC` when you click **Apply**, the same as with `local-ca`.

16. Click the **Key Usage** tab and check only **tls server**.
17. Click **Apply**.
18. Click **Sign** in the **Actions** list. The same error appears: `Error in Certificate - Selection expected!` Click **Cancel**, then **OK**.
19. In the Terminal, type this and press **Enter**:

```
/certificate/sign radius-server ca=local-ca
```

20. The Terminal prints `progress: done`. In the certificate list, `radius-server` now shows the flags **KI**.

### 3.4 Turn on HTTPS

21. Click **IP** in the left menu, then **Services**.
22. Double-click **www-ssl**.
23. Check **Enabled**, set **Certificate** to `ssl-web-config`, and click **Apply**. The **DISABLED** and **INVALID** badges at the bottom of the dialog disappear.
24. On your laptop, open `https://192.168.88.1` in a browser.
25. Your browser shows a certificate warning. That's expected: the router signed its own certificate, so your laptop has no reason to trust it. Click through the warning to continue.
26. Wait for the router's login page to appear.

    > **Note:** The first time you load this page, it can take about 30 seconds. Don't refresh or close the tab. After that, it loads almost instantly.

### 3.5 Reach your router over the network

27. **Save your Lab Notes first** (see Lab 0). Then unplug the Ethernet cable from your laptop. Your laptop is now reaching the class only over Wi-Fi.
28. Open a browser and go to `https://203.0.113.x`, replacing `x` with the last number of your WAN address from **Lab Notes**.
29. Click through the certificate warning. The router's login page loads, and the first load should be quick.

    > **Why:** Until now you've reached your router through the backdoor cable. This proves it also answers over the network.

30. Open WinBox. Your router won't appear in **Neighbors**, because discovery doesn't work over the WAN address. In the **Connect to** field, type your WAN address from **Lab Notes**.
31. Check that **Login** says `admin`. WinBox may have saved your old password, so clear **Password** and type your new one, then click **Connect**.
32. Look at the WinBox title bar. It now shows your WAN address instead of `192.168.88.1`.

> ### ⚠️ STOP AND READ
> Reaching WinBox and HTTPS through the WAN address is fine for a lab router. If you ever make this router your primary firewall, don't leave these open to the internet. Connect through a WireGuard tunnel and manage it from inside.

33. Plug the Ethernet cable back into **ether7**. In WinBox, click **Disconnect**, then click **Refresh** on the right side of the **Neighbors** list. Find your router by its identity, click its **IP address** (`192.168.88.1`), and log in with your password.

    > **Note:** WinBox won't see the cable until you click **Refresh**. If your router isn't listed, click it again.

    > **Why:** `192.168.88.1` lives on the cable. Your Wi-Fi has no route to that address, so your laptop sends WinBox traffic down the wire, and the backdoor works even while Wi-Fi is on. From here on, use the backdoor connection.
