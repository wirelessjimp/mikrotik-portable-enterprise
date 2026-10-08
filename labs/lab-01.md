# Lab 1 — Connect Your Router and Laptop

*Prerequisites: Lab 0*

**Why:** Before you configure anything, you need a reliable way into the router. This lab connects you through a dedicated backdoor port, so a bad configuration later can't lock you out.

### 1.1 Power and cable

1. Plug the power cable into the power strip, then into the barrel jack on the front left of the router.
2. Plug the 1 meter Ethernet cable from your Ethernet adapter into **ether7** on the router, and make sure your laptop's wired connection is set to DHCP.

   > **Note:** This port is known as the **backdoor port** from here on. It isn't a MikroTik concept. This is a *Jim has been locked out of way too many routers, so he added one* concept.

### 1.2 First login

3. Open WinBox, click the **Neighbors** tab, and wait about 30 seconds for your router to appear.
4. Find the row with Identity **Student-Lab**, IP **192.168.88.1**, and Version **7.24.5**. Click its **IP address**.

   > **Note:** You'll also see **Class-Router**, possibly twice. That's the instructor's router, not yours. Don't select it.

5. Log in with **Login** `admin` and **Password** `password`, then click **Connect**.

### 1.3 Set your identity

6. Click **System** in the left menu, then **Identity**.
7. Set **Identity** to the name on the label at your seat (`Student01` through `Student12`), then click **Apply** and **OK**. The screenshots in this guide use `Student31`. Use your own label.
8. Check the WinBox title bar and the bottom status bar. Both now show your new name.
9. Enter your identity in **Lab Notes**, in the **L009** column.

### 1.4 Change the password

10. Click **System** in the left menu, then **Password**.
11. In the **Change Password** dialog, enter the old password (`password`), then your new throwaway password in **New Password** and **Confirm Password**. Record the new one in **Lab Notes** first.
12. Click **Change**.
13. Click the **Disconnect** icon (the broken link) in the upper right of WinBox.
14. In **Neighbors**, find your router by its new identity and click its **IP address**. WinBox fills in your old password, so **clear the Password field** and type your **new** password, then click **Connect**.

> **Note:** If the login fails, the saved password is the usual cause. Clear the field and type the new one again.

### 1.5 Connect the WAN

15. Click **IP** in the left menu, then **Addresses**. Leave the window open.
16. Find the cable labeled **Router** and plug it into **ether1** on your main router, the L009.

    > **Note:** The cable labeled **mAP** isn't used yet. Leave it alone.

17. Watch the **Addresses** window. A new row appears at the bottom with a **D** flag, an address in `203.0.113.x`, and **ether1** as the interface.
18. Record that address in **Lab Notes**, in the **WAN IP Address** row under **L009**.
19. Look at the status bar at the bottom of WinBox. The date and time now match the real date and time.

    > **Note:** Your router gets its time from internet time servers (NTP), so it had no way to know the time until the WAN connected. Certificates depend on correct time, which is why we connected the WAN first.

### 1.6 Check the internet

20. Click **Tools** in the left menu, then **Ping**.
21. Set **Ping To** to `1.1.1.1`.
22. Click the **+** next to **Packet Count**, enter `4`, then click **Start**. The ping stops on its own.
23. The summary row should read **4 of 4 packets received** and **0 % packet loss**.

> **Check:** If you see packet loss or no replies, make sure the **Router** cable is in **ether1** on your L009 and look for the `203.0.113.x` row in **IP → Addresses**.
