# Lab 8 — RoMON (Manage a Device Without an IP Address)

*Prerequisites: Lab 6 (both devices reachable in WinBox), Lab 7*

**Why:** RoMON is a management overlay between MikroTik devices. It runs at Layer 2, so WinBox can reach a device that has no IP route to you. In this lab you give your two devices their own secret and open a session to your mAP by its ID, not its IP address.

> **Note:** The instructor's script turned RoMON on for every device in the room, including the class router, and gave them all the same secret. That's why your mAP was visible through the class router before you did anything. When you change the secret, your devices only talk to each other, so they need a direct cable between them.

### 8.1 Move the mAP back to your L009

1. Unplug the cable labeled **mAP** from the mAP's **ETH1**. Plug the 15 cm jumper into **ether8** on the L009 and into the mAP's **ETH1**. Wait about 30 seconds.
2. Your WinBox window to the mAP drops. Open a new one with the cube icon and connect to `10.255.255.2`. Clear **Password** first and type your password.

   > **Note:** The mAP's address on its WAN side changes back to `10.10.x.x`. The tunnel address `10.255.255.2` doesn't change, so that's the address to use.

### 8.2 Look at RoMON before you change anything

3. **L009 window:** click **Tools**, then **RoMON**. **RoMON Settings** shows **Enabled** checked, **Secrets** filled in, and a **Current ID** (a MAC address). Don't change anything yet.
4. Click **Discovery** under **Actions**. The list shows the class router and your mAP, each at **Hops** `1`, with the device's own ID as its **Path**. Click **Cancel** on **Discovery**.

   > **Why:** **Hops** counts the devices between you and the one listed. With the mAP on the class network, it showed `2` through the class router. On the jumper it's `1`.

### 8.3 Change the secret

5. Write a throwaway RoMON secret in **Lab Notes**, in the **RoMON** row under **Shared Secrets**.
6. **L009 window:** in **RoMON Settings**, **delete the existing secret** in the **Secrets** field, type your new one, and click **Apply**, then **OK**.
7. Open **Discovery** again. The list is empty.

   > **Why:** RoMON only talks to devices that share its secret. Your L009 no longer matches the mAP or the class router.

8. **mAP window:** click **Tools**, then **RoMON**. **Delete the existing secret**, type the same new one, and click **Apply**, then **OK**.
9. Click **Discovery** in the mAP window. It lists your L009 at **Hops** `1`. The class router isn't on the list.

   > **Why:** Your two devices share a secret, and the class router doesn't know it.

### 8.4 Open a session through RoMON

10. Click the cube icon in the L009 window. Set **Connect to** to `192.168.88.1`, clear **Password**, type your password, and click **Connect to RoMON**.
11. Click the **RoMON Neighbors** tab, click your mAP's row, and click **Connect**.
12. The title bar reads `admin@` followed by the mAP's **ID**, then `(Student31-mAP)`, then `via192.168.88.1`. The status bar at the bottom shows the same ID where an IP address used to be.

> **Why:** You didn't type an IP address to get in. WinBox found the mAP by its ID, over Layer 2, through your L009, and the title bar says which device carried the session.

> ### ⚠️ STOP AND READ
> After step 6, your L009 is cut off from the class router's RoMON until you change the mAP too. That's expected. If your mAP isn't on the jumper, it never shows up again, because it needs a direct cable once your devices have their own secret.
