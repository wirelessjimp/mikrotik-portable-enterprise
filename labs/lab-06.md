# Lab 6 — Backup and Configuration Export

*Prerequisites: Lab 5*

Before modifying bridges, VLANs, and firewall rules (Lab 7+), create a backup. This is your checkpoint — if something breaks, you can recover to this known-good state.

> **Context:** Up to this point, we've been building and adding. Starting with Lab 7, we begin modifying and removing default configurations. That's where things can break. Save your work now.

---

## Lab 6.1 — Creating a Binary Backup

Binary backups capture everything: configuration, certificates, user database, files. They're the "oh shit" recovery option.

### Create the Backup

1. Navigate to **Files** in the left menu

2. Under **Actions** on the right, click **Backup**

3. Configure:
   - **Name:** `pre-bridge-config` (descriptive names help when you have multiple backups)
   - **Encryption:** Don't Encrypt (for lab use; production backups should use encryption)

4. Click **Backup Config**

5. The backup file appears in the file list with a `.backup` extension.

### Download to Laptop

6. Click on the backup file to select it, then under **Actions** on the right, click **Download**

7. Save it to a known location on your laptop

### Copy to USB (Redundancy)

8. Drag and drop the backup file into the `usb1` folder in the Files list

9. Alternatively, download the backup to your computer (step 6-7), then upload it to the `usb1` folder using the **Upload** button

You now have backups in two locations.

> **Tip:** For ongoing work, use descriptive names (`pre-bridge-config.backup`). For production maintenance, use dates (`2026-03-14.backup`).

---

## Lab 6.2 — Restoring from a Binary Backup

If you need to restore:

1. Navigate to **Files**

2. Click on the backup filename

3. Click **Restore** under the right-hand **Action** menu

4. Confirm the correct file is selected

5. Enter password if the backup was encrypted

6. Click **Restore** and **OK**

The router reboots and restores the configuration (approximately 60 seconds).

> **Warning:** Binary backups are device-specific. Restoring to a different model may cause issues.

---

## Lab 6.3 — Exporting Configuration to RSC

RSC (RouterOS Script) exports are human-readable text files containing CLI commands that recreate your configuration. They're useful for:
- Version control (track changes in Git)
- Applying configuration to new devices
- Understanding what you've actually changed

### RSC vs Backup Files

A **backup file** (Lab 6.1) is a binary snapshot — it captures everything including passwords, certificates, and MAC addresses. It can only be restored to the same device and you can't read or edit it. Think of it as a full disk image.

An **RSC file** is a plain text script of CLI commands. You can read it, edit it, paste it into a different device, and track changes in Git. It doesn't include passwords, certificates, or device-specific data like MAC addresses. Think of it as a recipe — it tells a new device what to build, not what to clone.

Use backups for disaster recovery on the same device. Use RSC exports for documentation, version control, and deploying to new hardware.

### Full Export vs Compact Export

- **Full export** (`/export`) — Includes everything, even defaults. Can be hundreds of lines.
- **Compact export** (`/export compact`) — Only includes changes from defaults. Much cleaner.

For version control and documentation, **compact is preferred**.

### Create a Compact Export

1. Open a Terminal

2. Run:

   ```
   /export compact file=pre-bridge-config
   ```

3. This creates `pre-bridge-config.rsc` in the Files section.

### View the Export

4. Download the `.rsc` file to your laptop

5. Open it in a text editor (Notepad, VS Code, etc.)

You'll see the CLI commands that would recreate your current configuration. This is what you've built.

> **Note:** RSC files can be edited, version controlled, and applied to factory-reset devices. For full reset and import procedures, see **Appendix B — Reset and Recovery Procedures**.

---

## Lab 6.4 — When to Use Which

| Situation | Use |
|-----------|-----|
| Quick recovery to known state | Binary backup (Lab 6.1) |
| Track changes over time | RSC compact export + Git |
| Apply config to new device | RSC export + import on fresh device |
| Share config with someone else | RSC export (no passwords included) |
| Complete disaster recovery | Binary backup + See Appendix B |

> **Visual difference:** In the Files window, RSC exports show as type **script** and are typically small (under 10 KiB for a lab config). Binary backups show as type **backup** and are significantly larger because they include all device-specific data. You can see both side by side in your Files list.
> <img width="553" height="421" alt="image" src="https://github.com/user-attachments/assets/6952d7a3-a4c3-4090-9fc7-a6ffb2cbd0bb" />

---

Once you have the backups and RSC scripts saved to a safe place, you can proceed to Lab 07
