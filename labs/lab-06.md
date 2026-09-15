# Lab 6 — Backup and Configuration Export

*Prerequisites: Lab 5*

Before modifying bridges, VLANs, and firewall rules (Lab 7+), create a backup. This is your checkpoint — if something breaks, you can recover to this known-good state.

> **Context:** Up to this point, we've been building and adding. Starting with Lab 7, we begin modifying and removing default configurations. That's where things can break. Save your work now.

---

## Lab 6.1 — Creating a Binary Backup

Binary backups capture everything: configuration, certificates, user database, files. They're the "oh shit" recovery option.

### Create the Backup

1. Navigate to **Files** in the left menu

2. Under **Actions** on the right, click **Configuration**, then click **Backup**

3. Configure:
   - **Name:** `pre-bridge-config` (descriptive names help when you have multiple backups)
   - **Encryption:** Don't Encrypt (for lab use; production backups should use encryption)

4. Click **Backup**

5. The backup file appears in the file list with a `.backup` extension.

### Download to Laptop

6. Click on the backup file to select it, then click **Download**

7. Save it to a known location on your laptop

### Copy to USB (Redundancy)

8. Drag and drop the backup file into the `usb1` folder in the Files list

9. Alternatively, download the backup to your computer (step 6-7), then upload it to the `usb1` folder using the Upload button

You now have backups in two locations.

> **Tip:** For ongoing work, use descriptive names (`pre-bridge-config.backup`). For production maintenance, use dates (`2026-03-14.backup`).

---

## Lab 6.2 — Restoring from a Binary Backup

If you need to restore:

1. Navigate to **Files**

2. Click on the backup filename

3. Click **Restore**

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
| Quick recovery to known state | Binary backup (Lab 6.2) |
| Track changes over time | RSC compact export + Git |
| Apply config to new device | RSC export + import on fresh device |
| Complete disaster recovery | See Appendix B |

---

# Lab Notes Template

Print this page or copy to a document for recording important values.

---

**Lab 1 — Initial Configuration**

| Item | Value |
|------|-------|
| Router Identity | |
| Admin Password | |
| WAN IP Address | |
| Firmware Version | |

---

**Lab 2 — Storage and Packages**

| Item | Value |
|------|-------|
| Architecture | |
| RouterOS Version | |
| USB Drive Size | |
| Packages Installed | |

---

**Lab 3 — Management**

| Item | Value |
|------|-------|
| Device Mode | |
| RoMON Secret | |

---

**Lab 4 — WAN Access**

| Item | Value |
|------|-------|
| Management Network | |
| CA Certificate Name | |
| Web Certificate Name | |

---

**Lab 5 — Containers**

| Container | IP Address | Port | Status |
|-----------|------------|------|--------|
| OpenSpeedTest | 172.17.0.2 | 3000 | |
| iperf3 | 172.17.0.3 | 5201 | |
| nginx | 172.17.0.4 | 80 | |

| Resource Check | Free Memory |
|----------------|-------------|
| Before containers | |
| After OpenSpeedTest | |
| After iperf3 | |
| After nginx | |

---

**Backup Log**

| Date | Filename | Location | Notes |
|------|----------|----------|-------|
| | | | |
| | | | |
| | | | |

---

*Document Version: Draft 3.0*
*Last Updated: March 2026*
