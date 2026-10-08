# Lab 5 — User Manager (RADIUS Server)

*Prerequisites: Lab 3 (the `radius-server` certificate). The User Manager package is installed by the instructor before class.*

**Why:** User Manager is a RADIUS server that runs on your router. RADIUS checks a username and password for something else, in this class the mAP's enterprise Wi-Fi network. The instructor's script already built most of it, so you spend your time on the parts that are yours.

1. Click **User Manager** in the left menu. The window opens on **Routers**.
2. Look at the one row, `mikrotik-ap`. The address is `10.10.255.0/24`.

   > **Why:** A router entry lists the devices allowed to ask your router to check a login. `10.10.255.0/24` is the whole mAP management network. In a real network you'd list one address per access point. We opened the whole range so a typo doesn't cost you class time.

3. Click **User Groups**. Three groups are already there. The `*` flag marks the ones that came pre-built on the router.
   - `cert-auth` is for testing EAP-TLS (certificate logins). We don't use it yet.
   - `default` is the group your user is in. We changed it to accept only PEAP (username and password logins).
   - `default-anonymous` is also pre-built. We don't use it.
4. Click **Settings** under **Configuration** on the right. **Certificate** says `none`. Open the drop-down and choose `radius-server`, then click **Apply** and **OK**. **Apply** turns gray and nothing else changes.

   > **Why:** The drop-down lists every certificate on the router. `radius-server` is the one you built for this job.

5. Click the **Users** tab, then **New**. Set these in the order the window lists them:
   - **Name:** `user2@mikrotik.test`
   - **Password:** your own throwaway password. Record it in **Lab Notes** first, in **User Manager Accounts**.
   - **Group:** leave `default`
   - **Shared Users:** `3`

   Click **Apply**, then **OK**. The new user appears in the list.

   > **Why:** **Shared Users** is how many devices can log in with this account at once. Three covers a laptop, a phone, and a tablet.

6. Pick a throwaway shared secret and write it in **Lab Notes**, in the **RADIUS** row under **Shared Secrets**. Click the **Routers** tab and double-click `mikrotik-ap`. Type the secret into **Shared Secret**, then click **Apply** and **OK**.

   > **Note:** The mAP has no RADIUS client yet. You enter this same secret on the mAP in Lab 7.
