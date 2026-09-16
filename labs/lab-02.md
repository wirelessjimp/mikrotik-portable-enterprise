# Lab 2 — Storage, Packages, and Verification

*Prerequisites: Lab 1*

This lab prepares external storage, installs additional packages, downloads management tools, and verifies the router is ready for configuration.

---

## 2.1 — Prepare USB Storage

Containers and other features require storage beyond the router's internal flash. We'll format a USB drive now so it's ready when needed.

1. Insert a USB flash drive into the router's USB port.

2. Open a Terminal:
   - **WebUI:** Click the **Terminal** button in the upper right
   - **WinBox:** Click **New Terminal** in the left menu

   > **Note:** On first launch, the Terminal asks if you want to see the software license. Enter `n` to skip this prompt.

3. Verify the drive is detected:

   ```
   /disk print
   ```

   You should see the USB drive listed.

4. Format the drive with ext4 filesystem:

   ```
   /disk format usb1 file-system=ext4 mbr-partition-table=no
   ```

   > **CRITICAL:** You must use ext4, not fat32. Containers will not run on fat32-formatted storage.

   When prompted, click ```y``` to start the formatting

5. Verify formatting completed:

   ```
   /disk print
   ```

   Confirm the drive shows the ext4 filesystem.

---

## 2.2 — Identify Your Architecture and Version

6. Click **Advanced** in the upper right to switch from Terminal back to the WebFig interface.

7. Navigate to **System** → **Resources**

8. Note the following values:
   - **Architecture Name:** (e.g., `arm`, `arm64`, `x86`)
   - **Version:** (e.g., `7.22 stable`)

   You'll need both the architecture and exact version to download the correct packages.

---

## 2.3 — Download Packages and WinBox

9. In a new browser tab, navigate to **https://mikrotik.com/download**

10. Scroll down to the section where it starts **RouterOS**, select your options in this order:
    - **Architecture:** Select your architecture (e.g., ARM, ARM64, MIPSBE)
    - **Channels:** Select **Stable**
    - **Version:** Select the **exact version** that matches your router (from step 8)
    
    Under **EXTRA PACKAGES** click the **all packages** link to download the full package bundle.

    > **CRITICAL:** The package version must exactly match your installed RouterOS version. If you're running 7.22 but download 7.20.8 packages, they will fail to install silently. Check System → Packages to confirm your version before downloading.

    - **Wait for the download to complete before proceeding.**

11. Scroll back up and click **WINBOX** in the top menu bar, then select your operating system:
    - macOS (universal)
    - Linux (64-bit)
    - Windows (64-bit)

12. **Extract the downloaded package zip file on your laptop.**

---

## 2.4 — Upload Packages

13. Switching back to the MikroTik Router, in the MikroTik WebUI, click **Files** in the left menu.

14. Under **Actions** on the right, click **Upload**.

15. Upload the following packages (at minimum):
    - `container-[version]-[arch].npk`
    - `user-manager-[version]-[arch].npk`

    > **Tip:** If you can't see the `.npk` files, make sure you go to your downloads folder and unpack the zip file, see step 12 above.
    > **Tip:** You can upload additional packages from the bundle now. They won't activate until you reboot, and unused packages don't consume significant space.

17. Navigate to **System** → **Reboot** and click **Start** to install the packages.

18. After reboot, verify packages installed by navigating to **System** → **Packages**. You should see `user-manager` in the list, and in the left hand menu you should see a new menu item for `Container` near the bottom.

---

## 2.5 — Verify Time and DNS

Before building containers, verify the router has accurate time and working DNS. Both are required for pulling container images.

### Verify Time

18. Navigate to **System** → **Clock**

19. Confirm the date and time are approximately correct.

    > **Note:** By default, MikroTik syncs time via NTP from cloud.mikrotik.com or DHCP-provided servers. If time is significantly wrong, check your WAN connection. We'll cover detailed NTP configuration in Lab 23.

### Verify DNS

20. Open a Terminal and test DNS resolution:

    ```
    /ping google.com count=3
    ```

    If this resolves and pings successfully, DNS is working. Press `Ctrl+C` to cancel if you need to stop the ping early.

    > **Why this matters:** Containers pull images from the internet using domain names. If DNS fails, container deployment fails.

### If DNS Fails

If the ping command fails to resolve, add public DNS servers as a fallback:

```
/ip dns set servers=8.8.8.8,1.1.1.1
```

Then retry the ping test.

---

Once you have successfully completed all the steps here, feel free to move on to Lab 03.
