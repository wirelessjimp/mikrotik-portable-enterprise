# Lab 18 — RSC Files

*Prerequisites: Lab 0 (the portal and Lab Notes), Lab 1 (your L009), Lab 6 (your mAP, and a WinBox window for each device). Do this near the end, after the labs you plan to finish.*

**Why:** Your devices were built from two script files before class. Here you find out what a script file is, make one from your own device, run one, and see why a script behaves differently on an empty device than on a device that already has a configuration. That last part is why the completed files posted at the end of Day 2 work the way they do.

### 18.1 Back up both devices first

1. **L009 window:** click **Files**, then click **Backup** under **Actions**.
2. Set **Name** to `Student31-l009-before-rsc`. Use your own label (`Student01` to `Student12`). Leave the password blank, click **Don't Encrypt**, then click **Backup Config**.
3. Select `Student31-l009-before-rsc.backup` in the list and click **Download...** under **Actions**.
4. **mAP window:** do steps 1 to 3 again, with the name `Student31-mAP-before-rsc`. Download it now, not later. A backup has disappeared from the mAP's **Files** list after a restart before.
5. Open your **Downloads** folder and check that both files are there. On the instructor's router they were about 90 KB and 34 KB.

   > **Why:** Nothing in this lab is meant to break your devices. A backup is your way back if something does. It holds a device's whole configuration, including the Wi-Fi keys and passwords that an export leaves out.

> ### 🔐 Treat the backups like passwords
> They aren't encrypted, and each one holds a whole configuration. Delete both from your laptop when class ends (Lab 17.2).

### 18.2 What an RSC file is

An RSC file is a plain text file of RouterOS commands, one after another, the same commands you type in the Terminal. You've already used two. Before class, the instructor imported `class-student-l009.rsc` onto your L009 and `class-student-map.rsc` onto your mAP, both on empty devices. That is how your kit arrived with its bridges, DHCP servers, and firewall already in place.

| | Backup | RSC file |
|---|---|---|
| Extension | `.backup` | `.rsc` |
| Contents | A binary copy of one device's whole configuration | Plain text commands you can read and edit |
| Use it to | Put the same device back as it was | Build a device from a list, or build many from one list |

### 18.3 Make one from your own device

6. **L009 Terminal:** run this, with your own label:

```
/export file=Student31-l009-export
```

7. **mAP Terminal:** run this, with your own label:

```
/export file=Student31-mAP-export
```

8. In each window, click **Files**, select the new `.rsc` file, and click **Download...**. Download the mAP's file now.
9. Open both files in a text editor on your laptop. Don't save any changes. Each section starts with a line that begins with `/`, followed by `add` or `set` lines. Search each file for your label, such as `Student31`. You find the identity you set in Lab 1 and Lab 6.
10. Search the mAP's file for `wpa2-pre-shared-key`. Nothing matches, although your fallback network has a key.

   > **Why:** RouterOS leaves passwords, keys, and secrets out of an export on purpose, and it lists only the settings that differ from a device's defaults. A script built from an export needs its secrets put back by hand.

### 18.4 Run one

11. **L009 Terminal:** make a one-line script on the router itself, so no editor is involved, and look at it:

```
/file/add name=rsc-demo.rsc contents="/interface/bridge/add name=rsc-demo"
/file/print detail where name=rsc-demo.rsc
```

The list shows `type=script`, `size=35`, and your line under `contents`.

   > **Note:** `/file/get rsc-demo.rsc contents` prints nothing, even though the contents are there. Use `print detail`.

12. Run the script:

```
/import file-name=rsc-demo.rsc
```

It prints `Script file loaded and executed successfully`.

13. Check what it did:

```
/interface/bridge/print where name=rsc-demo
```

The list shows a bridge named `rsc-demo`, running. The import typed the command for you.

### 18.5 An empty device, and a device that already has a configuration

14. Run the same script again:

```
/import file-name=rsc-demo.rsc
```

It prints `Script Error: failure: already have interface with name rsc-demo (/interface/bridge/add; line 1)`. It names the command and the line.

   > **Why:** The script was written as if the bridge didn't exist. On an empty device that is true, and it works. On your device the bridge is already there, so the first line fails.

15. Now a script of two lines. The first fails, and the second would work:

```
/file/add name=rsc-demo-two.rsc contents="/interface/bridge/add name=rsc-demo\n/interface/bridge/add name=rsc-demo2"
/import file-name=rsc-demo-two.rsc
/interface/bridge/print terse where name~"rsc-demo"
```

The import fails on line 1 again. The list shows only `rsc-demo`. The second line never ran.

   > **Why:** An import stops at its first error. On a device that already has part of a configuration, a script stops at the first item that exists and skips everything after it. That leaves the device half built.

16. Now the same command, wrapped so a failure doesn't stop the script:

```
/file/add name=rsc-demo-wrapped.rsc contents=":do { /interface/bridge/add name=rsc-demo } on-error={ :log info \"rsc-demo: bridge exists or failed\" }"
/import file-name=rsc-demo-wrapped.rsc
/log/print where message~"rsc-demo"
```

The import prints `Script file loaded and executed successfully`. The log ends with `rsc-demo: bridge exists or failed`.

   > **Why:** `:do { ... } on-error={ ... }` runs the command in the first braces. If it fails, RouterOS runs the part in the second braces, here a log line, and carries on. The `\"` is how you type a quote inside the text of a command. The instructor's completed files wrap their blocks this way.

### 18.6 Clean up

17. **L009 Terminal:** remove the demo bridge and the three demo files:

```
/interface/bridge/remove [find name=rsc-demo]
/file/remove [find name~"^rsc-demo"]
/interface/bridge/print terse where name~"rsc-demo"
/file/print where name~"rsc-demo"
```

The last two print nothing.

### 18.7 What this means for the completed files

At the end of Day 2 the instructor posts `class-complete-l009.rsc` and `class-complete-map.rsc`. Each rebuilds a device as far as a script can. Part of each file is wrapped, so a block that already exists is logged and skipped. The first part of the L009's file is not wrapped, so it expects an empty device and stops at its first error. That is why Appendix A has you reset the device first. After an import, read the log to see what failed:

```
/log/print where message~"class-complete"
```
