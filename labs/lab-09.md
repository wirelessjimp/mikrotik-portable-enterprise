# Lab 9 — Client Certificates and EAP-TLS

*Prerequisites: Lab 3 (`local-ca`), Lab 5 (User Manager), Lab 7 (the enterprise SSID). The MikroTik app is installed on your phone before class.*

**Why:** PEAP uses a password. EAP-TLS uses a certificate on your phone instead, so there's no password to guess. You'll create a certificate for your phone, move it there, and join the same SSID with it. Steps 6 to 10 are the Android path. An iPhone needs a different route (section 9.5).

### 9.1 Create the client certificate

1. **L009 window:** click **System**, then **Certificates**, then **New**. On the **General** tab, set these in the order the window lists them:
   - **Name:** `user1-client`
   - **Common Name:** `user1@mikrotik.test`
   - **Digest Algorithm:** click the **+** and choose `sha384`
   - **Key Size:** `secp384r1`
   - **Days Valid:** `825`

   > **Note:** Leave **Subject Alt. Name** empty. **Key Type** changes from `RSA` to `EC` when you click **Apply**, as it did in Lab 3.

2. Click the **Key Usage** tab and check only **tls client**. Click **Apply**.
3. Sign it with `local-ca`. Click **Sign** in the **Actions** list, set **CA** to `local-ca`, and click **Start** once, or sign it from the Terminal:

```
/certificate/sign user1-client ca=local-ca
```

   When **Progress** reads `done`, the certificate window shows **PRIVATE KEY** and **ISSUED** at the bottom, and the list shows `user1-client` with the flags **KI**.

   > ### ⚠️ STOP AND READ
   > The **Sign** dialog sometimes fails with `Selection expected!`. It isn't your mistake. Use the Terminal command above.

4. Write a throwaway passphrase in **Lab Notes**, in the **Client Certificate Passphrase** row under **Shared Secrets**.
5. Double-click `user1-client` to reopen it, and click **Export** under **Actions**. Set these in the order the window lists them:
   - **Type:** `PKCS12`
   - **Export Passphrase:** click the **+**, then type the passphrase from **Lab Notes**
   - **File Name:** click the **+**, then type `user1-client`

   Click **Export**. Open **Files**. `user1-client.p12` is at the top level, about 1.5 KiB.

   > ### ⚠️ STOP AND READ
   > The `.p12` file holds your phone's private key. Delete it from **Files** when you're done.

### 9.2 Move it to your phone (Android)

6. Join `Student31-EAP` on your phone, using the PEAP login from Lab 7. Open the **MikroTik** app. On the **LOG IN** tab, enter `10.10.255.1`, **Login** `admin`, and your L009 password, then tap **CONNECT**. The first screen shows your L009 (`L009UiGS`, identity `Student31`).

   > **Why:** Your phone is on the mAP's Wi-Fi, and the mAP passes its traffic to the L009.

7. Tap **File Manager**, then `user1-client.p12`. The app offers **Download**, **Move**, and **Delete**. Tap **Download**, then **SAVE**. The phone says `Download completed`.
8. In the phone's **Settings**, search for `certificate` and open **Wi-Fi certificate**. Choose `user1-client.p12`.
9. At **Extract certificate**, type the passphrase from **Lab Notes**.
10. At **Name this certificate**, replace the long default name with `user1-client` and save it.

   > **Note:** These menu names are from one Android phone. Other versions and makers differ.

### 9.3 Create the matching user

11. **L009 window:** click **User Manager**, then the **Users** tab, then **New**. Set these in the order the window lists them:
    - **Comment:** `EAP-TLS test user`
    - **Name:** `user1@mikrotik.test`
    - **Group:** `cert-auth`

    Leave **Password** empty, and click **Apply**, then **OK**. Add the user to **Lab Notes**, in **User Manager Accounts**, with `cert-auth` in **Group**.

    > **Why:** The **Name** must match the certificate's **Common Name**. A certificate login has no password, so the dialog lets you leave it empty.

### 9.4 Join with the certificate

12. On your phone, forget `Student31-EAP` and join it again. Set **EAP method** to `TLS`, pick the `user1-client` certificate, set **CA certificate** to `Trust on First Use`, and enter `user1@mikrotik.test` as the identity. Tap **Connect**.
13. The phone asks you to confirm the server certificate again. Confirm it, and the phone joins. In **User Manager → Sessions**, a row for `user1@mikrotik.test` appears.

    > **Why:** One SSID serves both methods. User Manager decides by group. `user2` is in `default`, which allows PEAP only, and `user1` is in `cert-auth`, which allows TLS only.

### 9.5 iPhone path (needs a Mac)

**Why:** iOS only installs a `.p12` that holds one certificate and its private key. The `.p12` that RouterOS exports also carries `local-ca`, so the iPhone refuses it with `The container "Identity Certificate" must contain only one certificate and its private key.` Android doesn't mind the extra certificate. So you export the pieces separately and rebuild the `.p12` on a Mac.

14. **L009 window:** double-click your client certificate, click **Export** under **Actions**, and set **Type** to `PEM`. Click the **+** next to **Export Passphrase** and type the passphrase from **Lab Notes**. Click the **+** next to **File Name** and type `user1-client-pem`. Click **Export**.
15. Open **Files**. The export made two files, a `.crt` (about 660 B) and a `.key` (about 497 B). Select each one and click **Download...** under **Actions** to save it to your Mac's Downloads folder.
16. On your Mac, open **Terminal** and run this, with your own file names. It asks for the key's passphrase, then for a new export password twice. Write the new password in **Lab Notes**, in **Additional Notes**.

```
cd ~/Downloads
openssl pkcs12 -export -inkey user1-client-pem.key -in user1-client-pem.crt -name user1-client -out user1-client-ios.p12
```

17. AirDrop `user1-client-ios.p12` to the iPhone and tap it. A **Profile Downloaded** message appears. Open **Settings** and tap **Profile Downloaded**, then **Install**. iOS asks for your passcode. It then warns that the profile is not signed, so tap **Install**. Tap **Install** again at **Install Profile**. At **Enter Password**, type the new export password from the Mac and tap **Next**.
18. Forget `Student31-EAP` and join it again. Enter `user1@mikrotik.test` as the **Username**, and leave **Password** empty. Tap **Mode**, change it from `Automatic` to `EAP-TLS`, and tap back. Tap **Identity** and choose the certificate named `user1@mikrotik.test`. Tap **Join**. The phone shows the server certificate, `radius.mikrotik.test`. Tap **Trust**.
19. The iPhone joins and gets an address from the mAP, in `192.168.89.x`.

> ### ⚠️ STOP AND READ
> The `.key`, the `.crt`, and both `.p12` files hold your private key. Delete them from the router's **Files**, your Mac's Downloads folder, and the iPhone when you're done.

> **Note:** The iPhone shows the certificate under its Common Name. The **Not Signed** label on the profile is how iOS describes an installed `.p12`, not a problem. Your own iPhone may behave differently if it's managed by an employer.
