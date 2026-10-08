# Lab 2 — Meet the Terminal

*Prerequisites: Lab 1*

**Why:** Some steps in this class, starting with certificates, can only be done on the command line. Here you open the router's Terminal and learn how it differs from the terminal on your laptop. You also record the version and architecture of your router, and learn to find your way around the many windows WinBox opens.

1. Click **New Terminal** in the left menu. It opens in its own window.
2. Look at the prompt: `[admin@Student31] >`. The name in brackets is your router's identity.
3. Type `/system/identity/print` and press **Enter**. The router answers `name:` followed by the identity you set in Lab 1.
4. Click **System** in the left menu, then **Resources**.
5. Find **Version** and **Architecture Name**. In **Lab Notes**, record **Version** in the **RouterOS Version** row and **Architecture Name** in the **Architecture** row, both in the **L009** column. For this router they read `7.24.5 (stable)` and `arm`.

   > **Why:** The packages and containers you'll install later are built for one architecture. The L009 is `arm`, not `arm64`, and a package for the wrong one won't install.

   > **Note:** Ignore **Minimum Version**. It's not your RouterOS version.

6. In the Terminal, run `/system/device-mode/print`. Look at two lines. `mode` should read `advanced`, and `container` should read `yes`. If both do, skip to step 11.

   > **Why:** Containers are turned off in RouterOS until the router's device mode allows them, and Lab 4 builds three. RoMON (Lab 8) also needs `mode` to be `advanced`. Your instructor set this before class, but check it now, because a router that reads differently can't do Lab 4.

7. If either line reads differently, run:

```
/system/device-mode/update mode=advanced container=yes
```

   Run this whichever of the two lines was wrong.

8. The Terminal tells you to confirm the change within 5 minutes. Press the **mode button** on the router and hold it for a few seconds. The router reboots.

   > **Note:** The command alone changes nothing. Until you confirm, the old values stay.

9. Watch WinBox. It shows **Disconnected**, then logs you back in automatically. If it doesn't after a few minutes, reconnect the way you did in Lab 1.

10. Run `/system/device-mode/print` again. `mode` reads `advanced`, and `container` reads `yes`. If either still reads differently, tell your instructor.

11. Look at the top of WinBox, next to the **Workspace** drop-down. The number beside the window icon is how many windows are open. Click it to see the list, which shows each window with the menu path that opened it. Click a window to bring it to the front. **X** closes one window, and **Close All** clears the list.

> **Note:** The Terminal is the **router's** command line, not your laptop's. Commands typed here run on the MikroTik. Your laptop's Terminal or Command Prompt can't run them.
